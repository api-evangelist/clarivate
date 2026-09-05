---
name: clarivate-wos-journal-metrics
description: >-
  Retrieve Journal Citation Reports metrics — Journal Impact Factor, JCI, quartile and
  category rankings — for a journal or a Web of Science category, using the Web of
  Science Journals API. Use when evaluating where to publish, benchmarking a journal
  portfolio, or reporting on category performance for a JCR year.
api: Web of Science Journals API
spec: openapi/clarivate-wos-journal-openapi.json
base_url: https://api.clarivate.com/apis/wos-journals/v1
operations:
  - GET /journals
  - GET /journals/{id}
  - GET /journals/{id}/reports/year/{year}
  - GET /journals/{id}/cited/year/{year}
  - GET /journals/{id}/citing/year/{year}
  - GET /journals/{id}/history
  - GET /journals/history/title
  - GET /categories
  - GET /categories/{id}
  - GET /categories/{id}/reports/year/{year}
  - GET /categories/{id}/cited/year/{year}
  - GET /categories/{id}/citing/year/{year}
  - GET /last-updated
generated: '2026-09-05'
method: generated
source: >-
  https://developer.clarivate.com/apis/wos-journal/swagger — the 13 operations above
  are exactly those declared in the harvested contract. Clarivate declares no
  operationId on this API.
---

# Pull Journal Citation Reports metrics

## Before you start

- Access requires registration on the developer portal **and approval from Clarivate
  Sales/Product**. There is one plan, capped at **5 requests/second**.
- Auth: `X-ApiKey: <your key>`.
- Metrics are keyed by **JCR year**. The dataset refreshes once a year, in June —
  the 2024 release carried 2023 data. Call `GET /last-updated` first if freshness
  matters.

## Steps

1. **Find the journal.** `GET /journals?q=<name or ISSN>`

   Free-text search matches journal name, abbreviation, ISSN, publisher and (on
   `/categories`) category name. An ISSN must be a complete, valid pattern or it is
   not treated as an ISSN. The query must be at least three characters.

   Operators: space or `+` = AND, `OR`, `NOT` or `-` = exclude, and a trailing `*`
   wildcard (`q=nano*`).

2. **Narrow with filters.** Value filters — `edition`, `categoryCode`, `jcrYear`,
   `jifQuartile` — take one or more values separated by `;` (`categoryCode=RZ;RU`),
   and combine with `&`. `jcrYear` accepts exactly one year.

   Range filters — `jif`, `jifPercentile`, `jci` — take an operator prefix:
   `eq:` (not combinable), `gt:`, `gte:`, `lt:`, `lte:`. Combine two with `AND`:
   `jifPercentile=gte:50 AND lte:80`.

3. **Get the journal record.** `GET /journals/{id}`.

4. **Get the metrics for a year.** `GET /journals/{id}/reports/year/{year}`.

5. **Walk the citation network.** `GET /journals/{id}/cited/year/{year}` for journals
   citing it; `GET /journals/{id}/citing/year/{year}` for journals it cites. The same
   pair exists at category level.

6. **Check for title changes** before comparing across years:
   `GET /journals/{id}/history` and `GET /journals/history/title`.

## Pagination

`page` and `limit` (0-50, default 10); the response carries
`metadata{total, page, limit}`. There is no cursor.

## Error handling

Same envelope as the rest of the Web of Science estate:
`{"error": {"status", "title", "details"}}`, with 401 returning
`{"error_description", "error"}`. One code is specific to this API:

| Status | Title | Meaning |
|---|---|---|
| 404 | Suppressed Title | The journal was suppressed from JCR for that year. See https://jcr.help.clarivate.com/Content/title-suppressions.htm |

## Do not

- Do not compare a JIF across JCR years without checking `/journals/{id}/history` —
  titles merge, split and get suppressed.
- Do not pass more than one `jcrYear` value; the filter accepts a single year.
