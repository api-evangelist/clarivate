---
name: clarivate-wos-document-lookup
description: >-
  Verify a publication against the Web of Science Core Collection and retrieve its
  citation counts, using the Web of Science Starter API. Use when you have a DOI,
  title, ISSN or Web of Science UID and need authoritative bibliographic metadata or a
  times-cited figure.
api: Web of Science Starter API
spec: openapi/clarivate-wos-starter-openapi.json
base_url: https://api.clarivate.com/apis/wos-starter/v1
operations:
  - GET /documents
  - GET /documents/{uid}
  - GET /journals
  - GET /journals/{id}
generated: '2026-09-05'
method: generated
source: >-
  https://developer.clarivate.com/apis/wos-starter/swagger — operations verified
  against the harvested contract. Clarivate declares NO operationId on any Starter API
  operation, so operations are addressed by method + path.
---

# Look up a Web of Science document

## Before you start

- You need an API key. Register an application at
  https://developer.clarivate.com/help/application and subscribe it to the Starter API.
- The Free Trial Plan works without a Web of Science subscription but is capped at
  **1 request/second and 50 requests/day**, and it does **not** return times-cited.
  Times-cited requires the Free Institutional Member Plan (5/sec, 5,000/day).
- Send the key on every request: `X-ApiKey: <your key>`.

## Steps

1. **Search for the document.** `GET /documents?q=<query>&db=WOS&limit=10&page=1`

   `q` uses Web of Science field tags, not free text. The tags this API supports are
   TI (title), IS (ISSN/ISBN), SO (source title), VL, PG, CS, PY (year), AU (author),
   AI (author identifier), UT (accession number), DO (DOI), DT (document type),
   PMID, OG (organization), TS (topic) and SUR (DRCI source URL).

   Look up by DOI: `q=DO=10.1038/nature12373`
   Look up by title: `q=TI=(CRISPR gene editing)`

2. **Read the envelope.** The response carries `metadata.total`, `metadata.page` and
   `metadata.limit` alongside `hits[]`. `limit` accepts 0-50 and defaults to 10; page
   through with `page`. Stop when `page * limit >= metadata.total` — there is no
   cursor and no `has_more` flag.

3. **Fetch the full record.** `GET /documents/{uid}` with the `uid` from the hit
   (the Web of Science accession number, e.g. `WOS:000123456700001`).
   Add `detail=full` for the complete payload or `detail=short` for the abbreviated
   one that matches the retired Links AMR shape.

4. **Resolve the journal if you need journal-level context.**
   `GET /journals?q=<issn>` then `GET /journals/{id}`. For Journal Impact Factor and
   category metrics use the separate Journals API, not this one.

## Incremental sync

Use `modifiedTimeSpan=yyyy-mm-dd+yyyy-mm-dd` (or `publishTimeSpan`) to pull only
records changed in a window. This is the parameter Clarivate added specifically for
recurring data-integration jobs — use it instead of re-walking the full result set.

## Error handling

Clarivate returns `{"error": {"status", "title", "details"}}` — **not** RFC 9457
problem+json — with one exception: 401 returns
`{"error_description": "...", "error": "invalid_request"}`, and a rejection at the
gateway (before your request reaches the API) returns
`{"message": "No API key found in request", "request_id": "..."}`.

| Status | Meaning | What to do |
|---|---|---|
| 400 | Request syntax error | Check the field-tag syntax in `q` |
| 401 | Key invalid or missing | Confirm `X-ApiKey`; the key is bound to one application |
| 404 | Not found | The record may not be in the databases your plan covers |
| 405 | Method not allowed | This API is GET-only |
| 50X | Server error | Back off and retry; check https://status.clarivate.com/ |

## Throttling

There is **no rate-limit response header and no documented 429 body**. You cannot
learn your remaining budget from a response — enforce the published ceiling
client-side (1/sec on Free Trial, 5/sec on the institutional plans) and back off on
any 50X.

## Do not

- Do not retry a failed write — there are none here; this API is read-only.
- Do not assume `db=WOS` covers everything: `db` accepts WOS, BIOABS, MEDLINE, BCI,
  INSPEC, ZOOREC and DRCI, and your plan governs which you may query.
