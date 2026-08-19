---
name: spruce-health-schedule-and-manage-messages
description: >-
  Queue Spruce Health messages for future delivery — appointment reminders, follow-ups,
  recalls — list and cancel them before they send, and confirm delivery through the
  scheduled-message webhook events.
api: spruce-health:spruce-health-scheduled-messages
generated: '2026-08-15'
method: generated
source: openapi/_original/spruce-health-openapi.json
operations:
  - ScheduleConversationMessage
  - ListConversationScheduledMessages
  - ListScheduledMessages
  - DeleteScheduledMessage
  - ListSavedMessages
  - ContactConversations
  - CreateConversation
  - UploadMedia
---

# Schedule and manage Spruce Health messages

Base URL `https://api.sprucehealth.com/v1`, `Authorization: Bearer <organization-token>`.

Scheduled messages are **conversation-scoped** — you schedule *into a thread*, not at a
contact. Get the conversation first.

## 1. Get a conversation to schedule into

- `ContactConversations` — `GET /contacts/{contactId}/conversations` — existing threads for
  a contact.
- `CreateConversation` — `POST /conversations` — if none is suitable. `type: "secure"` for
  a patient thread (all participants must be patients with a Spruce account or a pending
  invite); `type: "note"` for an internal-only thread.

## 2. Reuse the practice's approved wording

`ListSavedMessages` — `GET /savedmessages` — the organization's message templates, private
and shared. Using these keeps automated sends in the practice's own voice rather than
inventing copy. Templates can carry attachments.

For new attachments, `UploadMedia` — `POST /media` — returns a reusable media id.

## 3. Schedule it

`ScheduleConversationMessage` — `POST /conversations/{conversationId}/scheduledmessages`.

- `scheduledToSendAt` — RFC 3339, when it should go.
- `sendAsInternalMemberId` — optional. **If no author is specified the message is sent as
  the organization**, not as a person. For a reminder that is usually right; for a
  clinician follow-up it usually is not.
- `isInternalNote` — schedule an internal note rather than a patient-visible message.

Send `s-idempotency-key`. A nightly reminder job that retries without one will
double-book the patient's inbox; with one, the duplicate is rejected with **422** and,
on a retry, that 422 means the first schedule succeeded.

## 4. Review the queue

- `ListScheduledMessages` — `GET /scheduledmessages` — everything pending for the
  organization. This is the operational view: what will go out tonight.
- `ListConversationScheduledMessages` — `GET /conversations/{conversationId}/scheduledmessages`
  — pending for one thread. Check this before scheduling another, so a patient who already
  has a reminder queued does not get two.

## 5. Cancel before it sends

`DeleteScheduledMessage` — `DELETE /scheduledmessages/{scheduledMessageId}` — removes a
scheduled message *before* it is sent. Once sent it is a conversation item and this
operation no longer applies — cancel it when the appointment is cancelled or rescheduled,
or the patient gets a reminder for a visit that is not happening.

## 6. Confirm delivery

Since **2026-07-31** scheduled messages have their own webhook events — subscribe rather
than poll:

| event                      | meaning                                        |
| -------------------------- | ---------------------------------------------- |
| `scheduledMessage.created` | a message was scheduled                        |
| `scheduledMessage.updated` | a scheduled message was modified               |
| `scheduledMessage.deleted` | cancelled before it was sent                   |
| `scheduledMessage.sent`    | it reached its send time and posted            |

`scheduledMessage.sent` is the one to act on. The scheduled message record also carries
**`sentConversationItemId`**, the forward pointer to the conversation item that was
actually created — that is how you tie a queued reminder to the message the patient saw,
and how you pick up their reply.

## Rules

- Rate limits are **per organization**; a bulk scheduling job competes with the practice's
  other integrations. Pace off `s-ratelimit-remaining` and `s-ratelimit-daily-remaining`.
- Errors are `{ "statusCode", "type", "message" }`. 404 on delete means it already sent or
  was already cancelled.
- Note conversations accept internal messages only.

See `asyncapi/spruce-health-webhooks.yml` and `conventions/spruce-health-conventions.yml`.
