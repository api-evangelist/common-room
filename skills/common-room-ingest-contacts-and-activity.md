---
name: Common Room ingest contacts and activity from your own source
description: Push people and their behaviour into Common Room from a system Common Room does not natively integrate with, using the v1 source-scoped upsert operations.
api: openapi/common-room-contacts-api-openapi.yml
operations:
  - "addUpdateUserToSource"
  - "addUpdateActivityToSource"
  - "getActivityTypes"
  - "POST /data-available"
generated: '2026-08-13'
method: generated
source: openapi/_original/common-room-core-openapi.yml + https://www.commonroom.io/docs/signals/custom-integrations/zapier-api/
---

# Ingest contacts and activity from your own source

Use this when the signal lives somewhere Common Room has no native integration
for — an internal app, a support tool, an event platform. Base URL is
`https://api.commonroom.io/community/v1`.

## Auth

`Authorization: Bearer <token>`. Confirm with `GET /api-token-status` before a
batch run; a `403` here means the batch will fail on every record.

## Steps

1. **Get your destination source id.** The `destinationSourceId` path parameter
   identifies the custom source the data lands in. It comes from the Common Room
   UI (Settings → Signals and integrations), not from the API. Store it in config,
   not in code.

2. **Learn the activity vocabulary.** `getActivityTypes` (`GET /activityTypes`)
   returns the activity types the workspace accepts. Map your internal event names
   onto these before sending anything — an unmapped type is a silent quality
   problem, not an error.

3. **Upsert the person.** `addUpdateUserToSource`
   (`POST /source/{destinationSourceId}/user`). This is "Add or Edit User" — it
   upserts. Send the strongest identifier you have; email and LinkedIn URL are the
   dedup keys.

4. **Upsert the activity.** `addUpdateActivityToSource`
   (`POST /source/{destinationSourceId}/activity`). Send the person's identifier,
   the mapped activity type, the content, and a real timestamp. Do the person
   first — an activity for an unknown person creates an orphan.

5. **Signal a bulk drop.** If you are landing data through the warehouse path
   (S3 / BigQuery / Redshift / Snowflake) rather than record-by-record, call
   `POST /data-available` (v2) once the drop is complete so Common Room imports it.

## Conventions that will bite you

- **The person upsert is idempotent; the activity upsert is not, in practice.**
  Re-sending the same person converges on one record. Re-sending an activity can
  append another timeline entry — key your job off your own source-of-truth
  cursor, not off retries.
- **Two responses mean two things.** These endpoints return `202` (accepted for
  processing) as well as `200`. A `202` is not a confirmation the record is
  queryable yet — do not immediately read it back and treat absence as failure.
- **Do not use the deprecated read path to verify.** `GET /user/{email}` and
  `GET /members` are marked deprecated in the published spec. Verify with the v2
  `GET /contacts` instead.
- **Rate limits are per token.** Batch jobs are the workload that hits them.
  Watch `X-RateLimit-Remaining` on every response and pause on `Retry-After`
  rather than waiting for the `429`.
- **PII in, PII out.** If a person exercises a deletion right, the removal path is
  `DELETE /user/{email}` (Right to be Forgotten) — and your ingest job must stop
  re-adding them, or you will resurrect the record on the next run.

## Output

Counts of people upserted, activities upserted, activity types that failed to map,
and any `429` pauses taken.
