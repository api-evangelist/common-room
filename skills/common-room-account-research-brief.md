---
name: Common Room account research brief
description: Assemble a grounded intelligence brief on one company — firmographics, lead scores, the people who matter, their recent activity, and web-visit signal — from the Common Room v2 API.
api: openapi/common-room-organizations-api-openapi.yml
operations:
  - "GET /organizations"
  - "GET /organizations/{id}"
  - "GET /contacts"
  - "GET /activities"
  - "GET /website-visits"
generated: '2026-08-13'
method: generated
source: openapi/_original/common-room-v2-openapi.yml + conventions/common-room-conventions.yml + errors/common-room-problem-types.yml
---

# Account research brief

Produce a brief on one company. Every fact must come from a Common Room response —
never fill a gap from model memory.

## Auth

Send `Authorization: Bearer <token>` on every call. Tokens are created by a room
Admin under Settings → API tokens in https://app.commonroom.io/. Base URL is
`https://api.commonroom.io/community/v2`.

Verify the token first with `GET /api-token-status`. A `403` with
`{"status":"forbidden"}` means the token is missing, invalid, or lacks access —
stop, do not retry.

## Steps

1. **Resolve the organization.** `GET /organizations?primaryDomain=<domain>`.
   Prefer the domain over the name — domain is the dedup key for organizations.
   If nothing comes back, the company is not in the workspace: fall through to
   `GET /prospector-companies?query=<name>` for net-new firmographics and say
   plainly in the brief that this is prospect data, not workspace data.

2. **Pull the full record.** `GET /organizations/{id}` with
   `cols=about,employees,revenueRangeMin,revenueRangeMax,subIndustry,location,leadScores,tags,customFields,recentNews,recentJobOpenings,surgingTopics,topContacts,researchResults`.
   Expensive columns are opt-in — a bare call returns a thin record.

3. **Get the people.** `GET /contacts?organizationId=<o_id>&limit=50&sort=latest_activity&direction=desc`
   with `cols=fullName,title,primaryEmail,leadScores,recentActivities,segments,tags,sparkSummary`.
   Sort by `latest_activity` so the most engaged contacts lead.

4. **Get the signal.** For the top contacts, `GET /activities?contactId=<c_id>&limit=25&sort=latest_activity&direction=desc`.
   Then `GET /website-visits?organizationId=<o_id>&limit=25` for anonymous and
   identified web activity.

5. **Name the sources.** Activities carry `providerId` / `providerName`. Resolve
   unknown providers once with `GET /providers` and cache the map for the session.

## Conventions that will bite you

- **IDs are prefixed and load-bearing.** `o_` organization, `c_` contact,
  `s_` segment, `ls_` lead score, `cf_` custom field. Passing a bare number
  returns `invalid_organization_id`, not `org_not_found`.
- **Pagination is cursor-based.** `limit` is 1–200, default 50. Read `nextCursor`
  from the response and pass it back as `cursor`. Stop when `nextCursor` is
  absent — never guess an offset.
- **Ask for a total explicitly.** Add `recordCount` to `cols` to get
  `meta.recordCount`; otherwise there is no total.
- **Errors are not RFC 9457.** The envelope is
  `{"success": false, "error": {"code": "...", "message": "..."}}`. Branch on
  `error.code`, not on the message string. See `errors/common-room-problem-types.yml`.
- **Respect the rate limit.** Every response carries `X-RateLimit-Limit`,
  `X-RateLimit-Remaining`, `X-RateLimit-Reset` and `Retry-After`. On `429`, read
  `rateLimit.waitMs` from the body and sleep before retrying. Every step in this
  skill is a GET, so retrying is always safe.

## Output

A brief with: company snapshot, lead score, top contacts ordered by engagement,
recent activity by source, web-visit signal, and open questions. Cite the object
ID behind every claim so a human can verify it.
