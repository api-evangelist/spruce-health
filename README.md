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
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Spruce Health is a HIPAA-compliant healthcare communication platform that unifies phone, SMS, secure messaging, video, e-fax, team chat, mobile payments and VoIP phone lines into one system for medical practices. The **Spruce Public API** is a RESTful, Bearer-token-authenticated interface (base `https://api.sprucehealth.com/v1`) spanning **47 operations** across contacts, conversations, conversation items, internal endpoints and phone lines, media, organization members and teams, saved and scheduled messages, AI transcriptions, and webhook endpoint management.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/spruce-health/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/spruce-health/refs/heads/main/apis.yml)

## Access Model (Important)

API access to Spruce is **gated twice**, not self-serve:

- The API is part of the **Communicator plan** ($49/user/month). The Basic plan ($24/user/month) does **not** include API access.
- Beyond having the Communicator plan, an organization must **contact Spruce Support to request that API access be enabled** ([request form](https://sprucehealth.com/spruce-api)).
- Once enabled, an administrator generates a token in the **API Access** section of Settings and sends it as `Authorization: Bearer <your-token>`. An incorrect or disabled token returns `403` — Spruce never returns `401`.

There is no separate API fee, and no sandbox or test mode: production is the only published environment.

The developer documentation, the API reference and the **machine-readable OpenAPI** are all publicly readable at [developer.sprucehealth.com](https://developer.sprucehealth.com/docs/overview), but calling the API requires an enabled, token-bearing organization.

## Contract Provenance

The OpenAPI in this repository is **the real one Spruce publishes**, harvested verbatim on 2026-08-15. Spruce's developer portal runs on ReadMe and references its definition as `oasPublicUrl: @spruce/v1.0#13needamst2v4m6`; the document is served from the ReadMe registry at `https://dash.readme.com/api/v1/api-registry/13needamst2v4m6` (HTTP 200, OpenAPI 3.0.0, `info.title` *Spruce Health API*, `servers[0]` `https://api.sprucehealth.com/v1`, 47 operations, 169 schemas).

It is saved unmodified at [`openapi/_original/spruce-health-openapi.json`](openapi/_original/spruce-health-openapi.json) and split by tag into 15 per-resource documents under [`openapi/`](openapi/).

> **Correction to an earlier round.** A previous pass could not find a published spec and authored an *honestly modeled* 38-operation OpenAPI from the operation catalog in `llms.txt`. That modeled spec has been **removed and replaced**. It had materially wrong paths — for example `/conversations/{conversationId}/items/{itemId}` (real: `/conversationItems/{conversationItemId}`), `/proxy-calls` (real: `/internalendpoints/{internalEndpointId}/calls`), `/contacts/{contactId}/integration-links` (real: `/integrationlinks`), and `/scheduled-messages` (real: `/scheduledmessages`) — and it omitted the Organization, Phone Lines, Teams, Transcription and contact-invite surfaces entirely. Every artifact derived from it has been regenerated from the real contract.

## What Spruce Actually Ships

| Surface | Status |
| --- | --- |
| OpenAPI 3.0.0, 47 operations | Published |
| Idempotency (`s-idempotency-key`, all POST/PATCH, 24h retention) | Published |
| Rate-limit response headers (4, across 60s and 24h windows) | Published |
| Request correlation id (`s-request-id`) | Published |
| Cursor pagination (`pageSize` + `paginationToken`, `hasMore`/`totalCount`) | Published |
| Webhooks — 15 event types, HMAC-SHA256 signed | Published |
| `llms.txt` on both the docs and marketing hosts | Published |
| Status page (Atlassian Statuspage, API tracked as its own component) | [status.sprucehealth.com](https://status.sprucehealth.com) |
| Dated changelog with RSS | [sprucehealth.com/whats-new](https://sprucehealth.com/whats-new) |
| MCP endpoint | Deployed at `developer.sprucehealth.com/mcp`, **authorization-gated** |
| First-party SDK in any registry | **None** |
| AsyncAPI | **None** — webhooks documented in prose only |
| `/.well-known/*` (security.txt, OAuth metadata, agent card) | **None** — 404 on all four hosts |
| OAuth 2.0 / scopes | **None** — a single all-or-nothing organization token |
| Sandbox / test mode | **None** |

## Compliance

- **HIPAA** — every eligible organization automatically receives a signed Business Associate Agreement as part of the terms of service; no separate agreement, no opt-in, no extra fee.
- **42 CFR Part 2** — supported since 2026-07-09 via an updated BAA bringing formal Qualified Service Organization status.
- **SOC 2 Type II** — Spruce states it is audited annually. No trust centre, audit period or report request path is published.

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
- **Modified:** 2026-08-15

## APIs

Fifteen API entries, one per tag in the published OpenAPI:

| API | Operations | Reference |
| --- | ---: | --- |
| Contacts | 11 | [listcontacts](https://developer.sprucehealth.com/reference/listcontacts) |
| Conversations | 6 | [listconversations](https://developer.sprucehealth.com/reference/listconversations) |
| Webhooks | 6 | [listwebhookendpoints](https://developer.sprucehealth.com/reference/listwebhookendpoints) |
| Scheduled Messages | 4 | [listscheduledmessages](https://developer.sprucehealth.com/reference/listscheduledmessages) |
| Internal Endpoints | 3 | [internalendpoints](https://developer.sprucehealth.com/reference/internalendpoints) |
| Organization | 3 | [organization](https://developer.sprucehealth.com/reference/organization-1) |
| Contact Fields | 2 | [contactfields](https://developer.sprucehealth.com/reference/contactfields) |
| Contact Tags | 2 | [contacttags](https://developer.sprucehealth.com/reference/contacttags) |
| Conversation Item | 2 | [conversationitem](https://developer.sprucehealth.com/reference/conversationitem) |
| Conversation Tags | 2 | [conversationtags](https://developer.sprucehealth.com/reference/conversationtags) |
| Phone Lines | 2 | [phonelines](https://developer.sprucehealth.com/reference/phonelines) |
| Media | 1 | [uploadmedia](https://developer.sprucehealth.com/reference/uploadmedia) |
| Saved Messages | 1 | [listsavedmessages](https://developer.sprucehealth.com/reference/listsavedmessages) |
| Teams | 1 | [teammembers](https://developer.sprucehealth.com/reference/teammembers) |
| Transcription | 1 | [transcription](https://developer.sprucehealth.com/reference/transcription) |

## WebSocket Review

Spruce does **not** expose a documented public WebSocket API. Real-time delivery is done with **outbound webhooks** (HTTPS callbacks) for `contact.*`, `conversation.*`, `conversationItem.*` and `scheduledMessage.*` events — not a `wss://` transport. See [`review.yml`](review.yml) and [`asyncapi/spruce-health-webhooks.yml`](asyncapi/spruce-health-webhooks.yml).

## Common Properties

- [Website](https://sprucehealth.com)
- [Developer Portal](https://developer.sprucehealth.com)
- [Documentation](https://developer.sprucehealth.com/docs/overview)
- [Status Page](https://status.sprucehealth.com)
- [Changelog](https://sprucehealth.com/whats-new)
- [Pricing](https://sprucehealth.com/plans)
- [GitHub](https://github.com/sprucehealth)
- [LinkedIn](https://www.linkedin.com/company/spruce-health)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
