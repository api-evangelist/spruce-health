---
name: spruce-health-poll-for-new-conversation-activity
description: >-
  Detect new patient activity in a Spruce Health organization without webhooks, using the
  documented orderBy=last_message + startFrom cursor pattern, and tell inbound patient
  replies apart from outbound staff messages.
api: spruce-health:spruce-health-conversations
generated: '2026-08-15'
method: generated
source: >-
  https://developer.sprucehealth.com/reference/listconversations.md and
  openapi/_original/spruce-health-openapi.json
operations:
  - ListConversations
  - ConversationItems
  - Conversation
  - ConversationItem
  - Contact
  - SearchContacts
  - Transcription
---

# Poll Spruce Health for new conversation activity

Base URL `https://api.sprucehealth.com/v1`, `Authorization: Bearer <organization-token>`.

Webhooks are the better mechanism. Use this when you cannot host a public HTTPS receiver,
or to backfill after a receiver outage — Spruce gives up after 10 retries over ~20
minutes, so an outage longer than that needs a catch-up pass.

## The cursor pattern

`ListConversations` — `GET /conversations` — is ordered by **created date by default**,
which is the wrong axis for change detection: a months-old thread with a new reply will
not move. Switch axes:

1. Call with `orderBy=last_message`. `orderBy` is **required** — omitting it is a 400.
2. Read `lastMessageAt` from the **last conversation in the page** and save it.
3. Next poll: `orderBy=last_message` **and** `startFrom=<saved lastMessageAt>` (RFC 3339).
   You get back only threads with activity since then.

Parameters:

| parameter         | notes                                                              |
| ----------------- | ------------------------------------------------------------------ |
| `orderBy`         | required — `created` or `last_message`                             |
| `startFrom`       | RFC 3339; results at or after this value on the ordering field      |
| `pageSize`        | max 500                                                            |
| `paginationToken` | **cannot be combined with `startFrom`**                            |

That last constraint is the one that bites. Within a single poll, page with
`paginationToken` and *drop* `startFrom`; across polls, use `startFrom` and *drop* the
token. Do not carry both.

To walk the whole organization once, start with neither `paginationToken` nor `startFrom`
and follow `paginationToken` while `hasMore` is true.

## Inbound or outbound?

Knowing a thread changed is not enough — you need to know whether a *patient* wrote.

4. `ConversationItems` — `GET /conversations/{conversationId}/items` — with `startFrom` set
   to your last checkpoint. If any item has **`direction: "inbound"`**, a patient (or other
   external party) sent it. `outbound` is your own team.
5. `ConversationItem` — `GET /conversationItems/{conversationItemId}` — for a single item.

## Eventual consistency

Both `ListConversations` and `ConversationItems` are explicitly documented as
**eventually consistent** — new records may take a short time to appear. Two consequences:

- Overlap your window slightly (re-poll from a few seconds before your checkpoint) rather
  than advancing it exactly, or you will skip records that landed late.
- Make the handler idempotent, because that overlap will re-deliver items.

## Resolve the participant

6. `Conversation` — `GET /conversations/{conversationId}` — the thread, its
   `externalParticipants`, `associatedContactIds` and the `internalEndpoint` it runs over.
7. `Contact` — `GET /contacts/{contactId}` — for a saved contact.
8. `SearchContacts` — `POST /contacts/search` — where the participant is unsaved or
   ambiguous, pass the `externalParticipant` endpoint's `rawValue` to find candidates.

## Voicemails and call recordings

A phone-call item carries its payload under `event.data`:

- `event.data.recordings[0].signedUrl.url` — a **time-limited** download link with an
  `expiresAt`. Fetch it promptly; do not persist the URL as a durable reference. Requires
  call recording enabled for the organization.
- `Transcription` — `GET /transcriptions/{transcriptionId}` — full transcript text plus AI
  summarization, reached via the `transcriptionId` on the item. There is no list
  operation. **Transcriptions created before November 2025 are in a legacy store and
  return 404** with an explanatory message even though their ids still appear on items —
  handle that 404 as "unavailable", not as a bug.

## Pacing

Polling burns the organization's shared rate budget, which is enforced **per organization,
not per credential**. Read `s-ratelimit-remaining` (60s) and `s-ratelimit-daily-remaining`
(24h) from every response and back off before exhaustion — the daily window is the one
that catches aggressive pollers, and Spruce does not document the status returned when you
hit it.

See `conventions/spruce-health-conventions.yml` and `rate-limits/spruce-health-rate-limits.yml`.
