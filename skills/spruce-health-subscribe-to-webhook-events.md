---
name: spruce-health-subscribe-to-webhook-events
description: >-
  Register a Spruce Health webhook endpoint, verify the HMAC signature on every delivery,
  handle the 15 published event types, and read the delivery log when events stop
  arriving.
api: spruce-health:spruce-health-webhooks
generated: '2026-08-15'
method: generated
source: >-
  https://developer.sprucehealth.com/docs/webhooks-overview.md and
  openapi/_original/spruce-health-openapi.json
operations:
  - CreateWebhookEndpoint
  - ListWebhookEndpoints
  - WebhookEndpoint
  - ListWebhookEndpointEvents
  - ModifyWebhookEndpointPaused
  - DeleteWebhookEndpoint
  - Conversation
  - SearchContacts
---

# Subscribe to Spruce Health webhook events

Base URL `https://api.sprucehealth.com/v1`, `Authorization: Bearer <organization-token>`.
Webhooks are reachable only through the Public API, so they inherit its gate.

## 1. Stand up the receiver first

Spruce sends to **HTTPS only** and expects a **2XX within 5 seconds**. Acknowledge
immediately and process asynchronously — do your work after you have replied, never
before.

## 2. Register the endpoint

`CreateWebhookEndpoint` — `POST /webhooks/endpoints`:

```bash
curl --request POST \
  --url https://api.sprucehealth.com/v1/webhooks/endpoints \
  --header 'authorization: Bearer <api-token>' \
  --header 'content-type: application/json' \
  --data '{ "name": "Events Endpoint", "url": "https://example.com/webhooks" }'
```

**The response contains the signing secret, and this is the only time you will ever
see it.** `ListWebhookEndpoints` never returns it. Store it securely on creation or you
will have to delete the endpoint and register a new one.

Send `s-idempotency-key` — a duplicate returns 422 rather than a second endpoint.

## 3. Verify every delivery

Signature is in the **`X-Spruce-Signature`** header, base64-encoded, HMAC-SHA256 over the
raw request body with your endpoint secret. Spruce's own reference implementation:

```go
func verifySignature(endpointSecret []byte, r *http.Request) (bool, error) {
    signature, err := base64.StdEncoding.DecodeString(r.Header.Get("X-Spruce-Signature"))
    // ... HMAC-SHA256 the body with endpointSecret ...
    return hmac.Equal(h.Sum(nil), signature), nil
}
```

Compare in constant time (`hmac.Equal`). Verify against the **raw** bytes — re-serializing
the JSON first will change them and every signature will fail.

Note the scheme signs the body **only** — there is no timestamp or key id, so it offers no
replay protection. If replay matters to you, deduplicate on the event `id` yourself.

## 4. Handle the event envelope

```json
{ "object": "event",
  "type":   "contact.created",
  "eventTime": "2024-05-15T00:00:00Z",
  "data": { "object": { "object": "contact", "id": "entity_…", … } } }
```

`data.object` carries the **full current state** of the resource, not a delta, so you do
not need a follow-up GET to apply the change.

The 15 published types:

| group              | events                                                                        |
| ------------------ | ----------------------------------------------------------------------------- |
| `contact`          | `.created` `.updated` `.deleted` `.merged`                                     |
| `conversation`     | `.created` `.updated` `.deleted`                                               |
| `conversationItem` | `.created` `.updated` `.deleted` `.restored`                                   |
| `scheduledMessage` | `.created` `.updated` `.deleted` `.sent`  *(added 2026-07-31)*                 |

Treat an unrecognised `type` as a no-op rather than an error — Spruce has added events
four times in the last year and will again.

## 5. Handle ordering and retries

- Events are **sent** in order but **may not arrive** in order. Order on `eventTime`, and
  make handlers idempotent — do not assume `.created` precedes `.updated`.
- A non-2XX or a timeout is retried **every 2 minutes, up to 10 attempts**. After that the
  event is lost, so a receiver down for ~20 minutes drops events permanently. Backfill
  with `ListConversations` (`orderBy=last_message`, `startFrom`) after an outage.
- Delivery is capped at **1000 events per minute per endpoint**; beyond that events resume
  after the window rather than being dropped.

## 6. Correlate your own writes

Mutations complete asynchronously. Match the id your write returned against the
`requestID` on the event:

- `PostConversationMessage` → `requestID` → `conversationItem.created`
- `PostMessageFromEndpoint` → `RequestID` → `conversationItem.created`
- `CreateConversation` (with a message) → `postMessageRequestId` → `conversationItem.created`
- `DeleteConversationItem` → `requestId` → `conversationItem.deleted`

## 7. Resolve a conversationItem event to a patient

The event carries `conversationId`. Call `Conversation` — `GET /conversations/{conversationId}`
— for the thread. For threads with unsaved or ambiguous participants, pass the
`externalParticipant` endpoint's `rawValue` to `SearchContacts` to find candidate contacts.

If integration links are in place, the contact payload echoes `integrationLinks`, so you
can resolve straight to your own record with no extra call.

## 8. Operate the endpoint

- `ListWebhookEndpoints` — `GET /webhooks/endpoints` — inventory (no secrets).
- `WebhookEndpoint` — `GET /webhooks/endpoints/{endpointId}`.
- `ListWebhookEndpointEvents` — `GET /webhooks/endpoints/{endpointId}/events` — **the
  debugging tool.** 30-day retention, 20 per page with a pagination token. Each record
  exposes `attempts`, `attemptLimit`, `deliveryTime` and `nextAttemptTime`, so you can see
  exactly where a failing delivery sits without instrumenting your receiver.
- `ModifyWebhookEndpointPaused` — `POST /webhooks/endpoints/{endpointId}/paused` — pause
  before a deploy, resume after. Better than letting the retry budget burn down.
- `DeleteWebhookEndpoint` — `DELETE /webhooks/endpoints/{endpointId}`.

See `asyncapi/spruce-health-webhooks.yml` for the full catalog and delivery contract.
