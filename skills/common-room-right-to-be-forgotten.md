---
name: Common Room right to be forgotten
description: Execute a GDPR/CCPA erasure request against Common Room correctly — verify, anonymize, confirm, and stop the pipes that would re-create the record.
api: openapi/common-room-right-to-be-forgotten-api-openapi.yml
operations:
  - "DELETE /user/{email}"
  - "GET /contacts"
  - "GET /api-token-status"
generated: '2026-08-13'
method: generated
source: openapi/_original/common-room-core-openapi.yml + https://www.commonroom.io/security/ + conventions/common-room-conventions.yml
---

# Right to be forgotten

Common Room's own security page commits to "a programmatic integration to remove
all personally identifiable information" for GDPR and CCPA. This is that
integration. It is destructive and it is not reversible — treat it accordingly.

## Auth

`Authorization: Bearer <token>` against `https://api.commonroom.io/community/v1`.
The token must belong to a room Admin. Confirm with `GET /api-token-status`
first — running an erasure batch on a token that turns out to be read-limited
produces a partial, unauditable result.

## Steps

1. **Confirm the request is real and in scope.** An erasure request needs a
   verified subject. This skill does not decide that; a human does. Record who
   approved it and when, before any call.

2. **Find the record.** `GET /contacts?limit=200` filtered to the subject's
   identity, with `cols=fullName,primaryEmail,segments,tags,customFields`. Capture
   the `c_` id and the segments/tags the person belongs to — you will need them
   for the completion record, and they will be gone afterwards.

3. **Anonymize.** `DELETE /user/{email}` on the v1 Core API. The operation is
   titled "Anonymize Contact" and is tagged both `Contacts` and
   `Right to be Forgotten`.

4. **Verify.** Re-run the step-2 lookup. The contact should no longer be
   retrievable by that email. Do **not** verify with `GET /user/{email}` — that
   operation is deprecated in the published spec.

5. **Close the inbound pipes.** This is the step that gets skipped and it is the
   one that matters. Any source that created the record will re-create it:
   - custom ingest jobs using `addUpdateUserToSource` (see the ingest skill),
   - Zapier flows,
   - native integrations (Slack, GitHub, Discourse, Salesforce, HubSpot…),
   - the website tracking snippet's `window.signals.identify({ email })` call.

   Add the subject to your own suppression list before the next sync runs.

6. **Record it.** Log the subject identifier, the approval, the timestamp, the
   HTTP status, and the pipes suppressed.

## Conventions that will bite you

- **Erasure is by email, not by id.** The path parameter is the email address —
  URL-encode it. A person with several emails on the contact (`allEmails`) may
  need more than one call; check the `allEmails` array captured in step 2.
- **`404` is ambiguous.** It can mean already-erased or never-present. Compare
  against the step-2 result before reporting "no such person".
- **Errors are not RFC 9457.** Branch on `error.code`. `invalid_contact_id` is a
  malformed input; `contact_not_found` is a real absence.
- **Never retry blindly through a 429.** Read `rateLimit.waitMs`, sleep, retry the
  same subject. The operation is idempotent — a second erasure of an already
  erased subject is harmless — but a hammered erasure batch that skips subjects
  silently is a compliance failure.

## Output

One row per subject: identifier, approval reference, HTTP status, verification
result, pipes suppressed, timestamp.
