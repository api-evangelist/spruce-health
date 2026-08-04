# Spruce Health (spruce-health)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
