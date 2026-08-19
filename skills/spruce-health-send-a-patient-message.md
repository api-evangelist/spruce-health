---
name: spruce-health-send-a-patient-message
description: >-
  Send a message to a patient from a Spruce Health practice over the right channel —
  secure in-app message, SMS, email or fax — from the correct internal endpoint, with
  attachments, and confirm it actually posted.
api: spruce-health:spruce-health-internal-endpoints
generated: '2026-08-15'
method: generated
source: >-
  https://developer.sprucehealth.com/reference/postmessagefromendpoint.md and
  openapi/_original/spruce-health-openapi.json
operations:
  - InternalEndpoints
  - SearchContacts
  - ListContacts
  - PostMessageFromEndpoint
  - PostConversationMessage
  - ContactConversations
  - UploadMedia
  - OrganizationMembers
  - PostCreateProxyCall
---

# Send a patient message from Spruce Health

Base URL `https://api.sprucehealth.com/v1`, `Authorization: Bearer <organization-token>`.

The key idea: **channel is a property of the internal endpoint you send from**, not a
flag on the message. Pick the endpoint, and you have picked secure/SMS/email/fax.

## 1. Choose the internal endpoint

`InternalEndpoints` — `GET /internalendpoints` — lists every Spruce phone number, fax
number, email address and Spruce Link the organization can send from. Each carries a
`channel`:

| channel  | use for                                   |
| -------- | ----------------------------------------- |
| `secure` | encrypted in-app message to a Spruce contact |
| `phone`  | SMS to a raw phone number                 |
| `email`  | email to a raw address                    |
| `fax`    | outbound fax to a fax number              |

Take `endpoint.Id` — it becomes the `internalEndpointId` path parameter.

## 2. Resolve the recipient

- `SearchContacts` — `POST /contacts/search` — free-text or structured lookup; the way to
  find a contact by phone, email, tag or external integration id.
- `ListContacts` — `GET /contacts` — for enumeration rather than lookup.
- `ContactConversations` — `GET /contacts/{contactId}/conversations` — if you would
  rather post into an existing thread (see step 5).

## 3. Attach media, if any

`UploadMedia` — `POST /media` — `multipart/form-data`. Set `Content-Disposition`
(`form-data; name="media"; filename="image.png"`) and `Content-Type` on the multipart
field; they drive how the file renders and downloads in the app. The returned **media id
is reusable across many messages** — upload once, attach repeatedly, via `attachmentID`.

## 4. Send from the endpoint

`PostMessageFromEndpoint` — `POST /internalendpoints/{internalEndpointId}/conversations`.

Set exactly one destination. Sending more than one is an error.

- **Secure** — populate `destination.secureEndpoint` with `contactId`, `subject`, and a
  `deliveryMethod`:
  - `any_available_secure_conversation` *(default)* — try conversations from this
    endpoint, then any existing secure conversation, then create one.
  - `only_conversations_matching_internal_endpoint` — restrict to threads from this
    endpoint; create one if none exists.
  - `new_conversation` — always start a fresh thread.
- **SMS or email** — populate `destination.smsOrEmailEndpoint` with the number or address,
  from a `phone` or `email` channel endpoint respectively.
- **Fax** — populate `destination.faxEndpoint`. Note: **every outbound fax creates a new
  fax conversation**, so do not expect fax replies to thread.

Send `s-idempotency-key`. A duplicate within 24 hours is rejected with **422** — which,
on a retry, means the first send already went out. Getting this wrong double-texts a
patient.

## 5. Or post into a conversation you already have

`PostConversationMessage` — `POST /conversations/{conversationId}/messages` — when the
thread is known. `body` is a **list of elements**, each `type: text` or `type: page`.

- To page a teammate, call `OrganizationMembers` (`GET /organization/members`) for the
  member id, then post with `internal: true` and an element
  `{ "type": "page", "value": "<member-id>" }`.
- Note conversations accept **internal messages only** — you must set `internal: true`.

## 6. Confirm it posted — the send is asynchronous

The 200 does **not** mean the message was delivered. It returns a correlation id:

| operation                  | field in response      |
| -------------------------- | ---------------------- |
| `PostConversationMessage`  | `requestID`            |
| `PostMessageFromEndpoint`  | `RequestID`            |
| `CreateConversation`       | `postMessageRequestId` |

Match it against the `requestID` field of the **`conversationItem.created`** webhook
event to know the message actually posted. Without a webhook consumer, you have no
confirmation signal — polling `ConversationItems` is the fallback, and it is eventually
consistent.

## Placing a call instead

`PostCreateProxyCall` — `POST /internalendpoints/{internalEndpointId}/calls` — creates an
outbound proxy call from an internal endpoint to an external number and returns the proxy
number to dial. The practice's number is what the patient sees; the staff member's
personal number is never exposed.

## Rules

- Rate limits are **per organization**; watch `s-ratelimit-remaining` and
  `s-ratelimit-daily-remaining`. No documented 429.
- Errors: `{ "statusCode", "type", "message" }`. 403 across the board means the token is
  disabled.
- SMS travels over the public telecom network — Spruce's own guidance is that it is only
  HIPAA-appropriate where the patient understands the limitations, has been offered a
  secure alternative, and still prefers SMS. Prefer a `secure` endpoint when the content
  is clinical.

See `conventions/spruce-health-conventions.yml` and `asyncapi/spruce-health-webhooks.yml`.
