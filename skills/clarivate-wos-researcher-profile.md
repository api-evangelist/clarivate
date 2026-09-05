---
name: clarivate-wos-researcher-profile
description: >-
  Resolve a researcher to their Web of Science profile and pull their publications and
  peer-review record, joining ResearcherID to ORCID. Use for faculty reporting,
  reviewer vetting, or syncing an institutional CRIS with Web of Science author data.
api: Web of Science Researcher API
spec: openapi/clarivate-wos-researcher-openapi.json
base_url: https://api.clarivate.com/apis/wos-researcher
operations:
  - GET /researchers
  - GET /researchers/{rid}
  - GET /researchers/{rid}/documents
  - GET /researchers/{rid}/peer-reviews
generated: '2026-09-05'
method: generated
source: >-
  https://developer.clarivate.com/apis/wos-researcher/swagger — the four operations
  above are exactly those declared in the harvested contract. Clarivate declares no
  operationId on this API.
---

# Resolve a researcher and pull their record

## Before you start

- One plan only: **5 requests/second, 5,000 requests/day**, and it requires a Web of
  Science subscription.
- Auth: `X-ApiKey: <your key>`.

## Steps

1. **Find the researcher.** `GET /researchers?q=<query>`

   The contract documents these search fields, and ORCID is one of them:
   - `ORCID=0000-0000-000-0000`
   - `RID=A-1009-2008` (Clarivate ResearcherID)
   - `AU=` author name
   - organization-enhanced name search
   - `claimStatus=false` to filter unclaimed profiles
   - `CU~"england"` for country

   Searching by ORCID is the reliable join from an external system. Name search is
   not — Web of Science author records disambiguate imperfectly, so always confirm
   with an identifier before writing a match back into your own records.

2. **Get the profile.** `GET /researchers/{rid}` returns the detailed record. Since
   the 2024-06-20 release the search response already carries ORCID, so you do not
   need this call purely to resolve a RID to an ORCID.

3. **Pull publications.** `GET /researchers/{rid}/documents`.

4. **Pull peer reviews.** `GET /researchers/{rid}/peer-reviews` — the Publons-derived
   review record.

## Pagination

`page` and `limit` (0-50, default 10), with `metadata{total, page, limit}` in the
response. Budget your paging against the 5,000/day ceiling: a 1,000-author sweep at
`limit=50` is well inside it, the same sweep at the default `limit=10` is not.

## Privacy

Researcher profiles are personal data. Records include claim status, affiliations and
country. Handle under the terms you agreed at
https://clarivate.com/wp-content/uploads/dlm_uploads/2019/08/End-User-Terms.pdf and
do not redistribute profile data outside your institution's licence.

## Error handling

`{"error": {"status", "title", "details"}}`; 401 returns
`{"error_description", "error"}`. 405 on anything other than GET — this API is
read-only.

## Do not

- Do not infer an ORCID from a name match. Only accept an ORCID the API returns.
- Do not retry into the per-second ceiling; there is no rate-limit header and no
  documented 429 to tell you that you crossed it.
