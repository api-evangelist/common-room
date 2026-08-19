---
name: Common Room build and populate a segment
description: Create a static segment and get the right contacts or organizations into it, using the v2 create operation plus the MCP/CLI update path — and knowing which v1 route is deprecated.
api: openapi/common-room-segments-api-openapi.yml
operations:
  - "POST /segments"
  - "GET /segments"
  - "GET /segments/{id}"
  - "GET /contacts"
  - "GET /organizations"
  - "addContactsToSegment"
  - "getSegmentStatuses"
generated: '2026-08-13'
method: generated
source: openapi/_original/common-room-v2-openapi.yml + openapi/_original/common-room-core-openapi.yml + mcp/common-room-mcp.yml + cli/common-room-cli.yml
---

# Build and populate a segment

## Auth

`Authorization: Bearer <token>` against `https://api.commonroom.io/community/v2`.
The v1 operations referenced below live at
`https://api.commonroom.io/community/v1`.

## Steps

1. **Check for an existing segment first.** `GET /segments?limit=200`. Segment
   names are unique — creating a duplicate returns `409` with
   `error.code: conflict`. If one already exists, reuse its `s_` id.

2. **Create the segment.** `POST /segments` with a name and an entity type
   (`contact` or `organization`). The entity type is fixed at creation and decides
   what can go in it. Response is `201` carrying the new `s_` id.

3. **Find the members.** For a contact segment,
   `GET /contacts?limit=200&sort=latest_activity&direction=desc` plus whichever of
   `organizationId`, `segmentId`, `query` narrows it; add `cols=fullName,primaryEmail,title,leadScores,tags`
   so you can justify each inclusion. Page with `cursor` until `nextCursor` is
   gone. For an organization segment, the same shape on `GET /organizations`.

4. **Add the members.** There is no public v2 REST operation for segment
   membership. Two supported routes:
   - **MCP** — `commonroom_update_object`, setting `segmentId` on each contact or
     organization (`https://mcp.commonroom.io/mcp`, OAuth 2.1).
   - **CLI** — `cr contact update --contact-id c_8812 --segment-id s_123456`.

   The v1 `addContactsToSegment` (`POST /segments/{id}`) still exists but is marked
   **deprecated** in the published spec. Do not build new automation on it.

5. **Verify.** `GET /segments/{id}` and read `entityCount`. For a contact segment
   also check the workflow statuses with `getSegmentStatuses`
   (`GET /segments/{id}/status`, v1) if the team tracks progress with statuses.

## Conventions that will bite you

- **Idempotency is by natural key, not by request.** There is no
  `Idempotency-Key` header. Contact and organization creates upsert on email /
  LinkedIn URL / domain, so those are safe to repeat. Segment creation is **not** —
  a repeat returns `409 conflict`. Always do step 1.
- **Segment entity type is immutable.** Putting an organization in a contact
  segment is not a validation error you can recover from; it is the wrong segment.
- **Batch with `--dry-run` first if you are driving the CLI.** Every `create` and
  `update` accepts it and prints the exact payload without sending.
- **Backoff on 429.** `X-RateLimit-Remaining` drops fast on wide `GET /contacts`
  paging. Sleep `rateLimit.waitMs` and continue from the last `nextCursor`.

## Output

The segment id, the entity type, the final `entityCount`, and the list of ids you
added with the signal that justified each one.
