---
name: spruce-health-sync-contacts-from-external-system
description: >-
  Backfill and keep in sync the patient records of an EHR or practice management system
  as Spruce Health contacts, linked bidirectionally so either side can resolve the other
  without keeping its own mapping table.
api: spruce-health:spruce-health-contacts
generated: '2026-08-15'
method: generated
source: >-
  https://developer.sprucehealth.com/docs/integration-guide.md and
  openapi/_original/spruce-health-openapi.json
operations:
  - SearchContacts
  - CreateContact
  - UpdateContact
  - DeleteContact
  - ContactTags
  - CreateContactTag
  - ContactFields
  - CreateContactField
  - CreateContactIntegrationLink
  - ContactIntegrationLinks
  - DeleteContactIntegrationLink
---

# Sync contacts from an external system into Spruce Health

Base URL `https://api.sprucehealth.com/v1`. Every request carries
`Authorization: Bearer <organization-token>`. API access is gated — the organization must
be on the Communicator plan *and* have API access enabled by Spruce Support before a
token exists at all.

## Before you start: define the org-level vocabulary once

Tags and custom fields are defined on the organization, not per contact, so create them
once at setup and reuse the ids.

1. `ContactTags` — `GET /contacts/tags` — list existing tags.
2. `CreateContactTag` — `POST /contacts/tags` — create any that are missing. Creating a
   tag that already exists returns **201 with the existing tag**, not an error, so this
   step is safe to re-run.
3. `ContactFields` — `GET /contacts/fields` — list organization contact fields.
4. `CreateContactField` — `POST /contacts/fields` — create fields such as `MRN` or
   `Preferred Pharmacy`. Same forgiving 201-on-duplicate behaviour.

Stamp external-system metadata (MRN, insurance, primary care provider, account number)
into these fields so staff see it beside the conversation.

## Backfill and ongoing create

5. `SearchContacts` — `POST /contacts/search` — check whether the record already exists
   before creating it. This is a POST rather than a GET because Spruce takes nested
   structured filters in the body; `GET` with a body is not portable across HTTP
   libraries.
6. `CreateContact` — `POST /contacts`.
   - Minimum requirement: at least one of `givenName`, `familyName`, `phoneNumbers`,
     `faxNumbers`, `emailAddresses`.
   - **Always send `s-idempotency-key`.** Derive it deterministically from the external
     record id so a retry after a timeout cannot create a second contact. Max 255
     characters; Spruce keeps keys for 24 hours.
   - Attach `tagIds` and `organizationContactFields` in the same call.
   - The response returns the full contact including the Spruce id (`entity_…`).
     **Persist that id against the external record.**

## Link the two systems

7. `CreateContactIntegrationLink` — `POST /contacts/{contactId}/integrationlinks` with
   `type: "custom"`, your `externalId`, and a `url` that opens the record in your
   system. Staff get a deep link out of Spruce; you get a canonical external id back.

Once links exist you no longer need your own mapping table in the read direction:

8. `SearchContacts` with an `integrationIDFilter` resolves an external id straight to
   the Spruce contact:

   ```json
   { "structured": { "integrationIDFilter": [
       { "integrationIDs": [ { "integrationLinkType": "custom", "id": "ext_patient_12345" } ],
         "match": "any" } ] } }
   ```

   Every contact payload — including the one embedded in webhook events — echoes
   `integrationLinks` back, so a `contact.updated` event resolves to your record with no
   extra call.

9. `ContactIntegrationLinks` / `DeleteContactIntegrationLink` — list and remove links.
   Delete is by `type` + `externalId`, not by a link id.

## Ongoing updates and deletes

10. `UpdateContact` — `PATCH /contacts/{contactId}`. Omitted fields are unchanged; an
    empty string or empty array **clears** the field. That asymmetry is the one to be
    careful with — a serializer that emits `""` for a null will wipe data. Send
    `s-idempotency-key` here too.
11. `DeleteContact` — `DELETE /contacts/{contactId}`. Check the `canDelete` flag on the
    contact first; contacts involved in active conversations generally are not deletable
    and the call will fail with 400.

## Rules that apply to every step

- **Idempotency**: `s-idempotency-key` is accepted on all `POST` and `PATCH`. A duplicate
  key inside the 24-hour window returns **422**, and Spruce **rejects rather than
  replays** — unlike Stripe or Square. On a retry, treat 422 as *"the first attempt
  succeeded"* and read the record back to get its id. Do not treat it as validation
  failure.
- **Rate limits** are enforced **per organization, not per credential**, so extra tokens
  buy no headroom and your backfill will throttle the practice's other integrations.
  Pace off `s-ratelimit-remaining` (60s window) and `s-ratelimit-daily-remaining` (24h
  window). Spruce does not document the status returned on exhaustion, so do not wait for
  a 429 — it may never come.
- **Pagination**: `pageSize` (max 500) + `paginationToken`; loop while `hasMore` is true.
- **Errors** are `{ "statusCode", "type", "message" }`, not RFC 9457. A blanket 403 across
  every call means the token is wrong or API access was disabled — not a permission issue
  on one record.
- **Log `s-request-id`** from each response; it is the correlation id to quote to Spruce
  Support.

See `conventions/spruce-health-conventions.yml`, `errors/spruce-health-problem-types.yml`
and `data-model/spruce-health-data-model.yml`.
