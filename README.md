# Spruce Health (spruce-health)

Spruce Health is a HIPAA-compliant healthcare communication platform that gives modern clinics secure messaging, voice, video, team chat, e-fax, secure payments, and phone lines in one system. The **Spruce Public API** is a RESTful, Bearer-token-authenticated interface (base `https://api.sprucehealth.com/v1`) that lets an organization manage Contacts, Conversations, conversation items and Messages, internal endpoints and phone lines, and register Webhook endpoints for real-time events.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/spruce-health/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/spruce-health/refs/heads/main/apis.yml)

## Access Model (Important)

API access to Spruce is **gated**, not self-serve:

- The API is part of the **Communicator plan** ($49/user/month). The Basic plan ($24/user/month) does **not** include API access.
- Beyond having the Communicator plan, an organization must **contact Spruce Support to request that API access be enabled**.
- Once enabled, an administrator generates a token in the **API Access** section of Settings and sends it as `Authorization: Bearer <your-token>`. An incorrect or disabled token returns `403`.

The developer documentation and API reference are **publicly readable** at [developer.sprucehealth.com](https://developer.sprucehealth.com/docs/overview) (including a machine-readable index at [developer.sprucehealth.com/llms.txt](https://developer.sprucehealth.com/llms.txt)), but calling the API requires an enabled, token-bearing organization.

## Tags

- Healthcare
- HIPAA
- Communication
- Secure Messaging
- Telehealth
- Contacts
- Conversations
- Messaging
- Webhooks
- VoIP

## Timestamps

- **Created:** 2026-07-10
- **Modified:** 2026-07-10

## APIs

### Spruce Contacts API

List, search, create, retrieve, update, and delete the contacts (patients and other parties) in a Spruce organization, plus manage contact custom fields, contact tags, contact-to-conversation relationships, integration links to external systems (EHR/PM), and send Spruce invites.

- **Human URL:** [https://developer.sprucehealth.com/reference/listcontacts](https://developer.sprucehealth.com/reference/listcontacts)
- **Base URL:** `https://api.sprucehealth.com/v1`

### Spruce Conversations API

List, create, retrieve, and update conversations (message threads) in an organization, and manage conversation tags. Listing supports cursor pagination and ordering by creation time or last message.

- **Human URL:** [https://developer.sprucehealth.com/reference/listconversations](https://developer.sprucehealth.com/reference/listconversations)
- **Base URL:** `https://api.sprucehealth.com/v1`

### Spruce Messages API

Post messages into a conversation, list and retrieve conversation items (messages, calls, faxes, secure conversation events), delete a conversation item, upload media, send messages from an internal endpoint or phone line, create proxy calls, and manage scheduled and saved messages.

- **Human URL:** [https://developer.sprucehealth.com/reference/postconversationmessage](https://developer.sprucehealth.com/reference/postconversationmessage)
- **Base URL:** `https://api.sprucehealth.com/v1`

### Spruce Webhooks API

Register and manage webhook endpoints so an organization receives real-time HTTP callbacks for contact, conversation, and conversation-item events (created / updated / deleted / merged / restored). Create endpoints (which returns a signing secret), list them, retrieve one, list an endpoint's events, pause or resume dispatch, and delete an endpoint.

- **Human URL:** [https://developer.sprucehealth.com/docs/webhooks-overview](https://developer.sprucehealth.com/docs/webhooks-overview)
- **Base URL:** `https://api.sprucehealth.com/v1`

## Grounding & Modeling Note

The **base URL**, **Bearer authentication**, **gated access model**, **pricing**, and the following **six endpoints** are confirmed directly from Spruce's public developer reference:

- `GET /contacts`
- `GET /conversations`
- `POST /conversations/{conversationId}/messages`
- `GET /internalendpoints`
- `GET /webhooks/endpoints`
- `POST /webhooks/endpoints`

The remaining paths in [`openapi/spruce-health-openapi.yml`](openapi/spruce-health-openapi.yml) (individual CRUD, fields/tags, integration links, conversation items, media upload, proxy calls, scheduled/saved messages, and webhook get/delete/events/pause) are **honestly modeled** from Spruce's published operation catalog and documented behavior. The OpenAPI is flagged `x-endpointsModeled: true`; reconcile exact paths and schemas against the live reference and the machine-readable OpenAPI Spruce publishes.

## WebSocket Review

Spruce does **not** expose a documented public WebSocket API. Real-time delivery is done with **outbound webhooks** (HTTPS callbacks) for `contact.*`, `conversation.*`, and `conversationItem.*` events — not a `wss://` transport. See [`review.yml`](review.yml).

## Common Properties

- [Website](https://sprucehealth.com)
- [LinkedIn](https://www.linkedin.com/company/spruce-health)
- [Documentation](https://developer.sprucehealth.com/docs/overview)
- [Sign Up / Plans](https://sprucehealth.com/plans)
- [Plans](plans/spruce-health-plans-pricing.yml)
- [Rate Limits](rate-limits/spruce-health-rate-limits.yml)
- [Fin Ops](finops/spruce-health-finops.yml)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
