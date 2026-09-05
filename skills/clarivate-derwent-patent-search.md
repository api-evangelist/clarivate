---
name: clarivate-derwent-patent-search
description: >-
  Search and retrieve Derwent patent, trademark, design and case-law records through
  the Clarivate Intellectual Property (IP) Data API. Use for prior-art searching,
  freedom-to-operate work, competitor portfolio mapping, or pulling patent PDFs and
  images by identifier.
api: Intellectual Property (IP) Data API
spec: openapi/clarivate-ipdata-api-openapi.json
base_url: https://api.clarivate.com
operations:
  - derwentSearchByQuery
  - derwentSearchByIds
  - derwentSearchByPatentNumbers
  - derwentCombinedSearch
  - derwentClassSearch
  - derwentClassBrowse
  - derwentCorporateTree
  - derwentDocumentsByIds
  - derwentDocumentsByPns
  - derwentDocumentsByListref
  - getPublicationIdMapping
  - checkPatentDocumentAvailability
  - checkPatentAbstractAvailability
  - getPatentDocuments
  - getPatentDocument
  - getPatentPdf
  - getPatentImages
  - derwentGetDocumentsPdf
  - derwentGetDocumentImages
  - derwentPostDocumentImages
  - getMatchPatentEntities
  - getMatchPatentEntity
  - searchPatents
  - searchTrademarks
  - getTrademarkDocuments
  - getTrademarkDocument
  - searchCaselaw
  - getCaselawDocuments
  - getCaselawDocument
  - checkCaselawDocumentAvailability
generated: '2026-09-05'
method: generated
source: >-
  https://developer.clarivate.com/apis/ipdata-api/swagger — all 33 operationIds are
  declared verbatim in the harvested contract; the 30 above are the ones a search and
  retrieval workflow uses.
---

# Search Derwent IP data

## Before you start

- Access is contract-reviewed: "All requests are reviewed against your contract and
  use case(s) prior to being approved." There is **no published plan table and no
  published throttle** for this API.
- Auth: `X-ApiKey: <your key>`. Some image endpoints additionally reference JWT
  authentication in their descriptions — check the operation you are calling.
- Clarivate publishes a first-party Python client:
  https://github.com/clarivate/ipdata-api-py-client (GitHub source only; not on PyPI).

## Search

Most search operations are **POST with a query body** — POST is the query verb here,
not a write. None of them mutates state.

- `derwentSearchByQuery` — POST `/patents/derwent/search-by-query` — the general
  entry point for a Derwent query.
- `derwentSearchByPatentNumbers` — POST `/patents/derwent/search-by-pns` — when you
  already hold publication numbers.
- `derwentSearchByIds` — POST `/patents/derwent/search-by-ids` — when you hold
  document GUIDs.
- `derwentCombinedSearch` — POST `/patents/derwent/combined-search` — combine
  criteria in one call rather than intersecting client-side.
- `derwentClassSearch` / `derwentClassBrowse` — classification-driven search and
  browse.
- `derwentCorporateTree` — POST `/patents/derwent/corporate-tree` — resolve an
  assignee to its corporate tree before portfolio counting, or you will undercount
  subsidiaries.
- `searchPatents`, `searchTrademarks`, `searchCaselaw` — the analytic-format search
  across each content set.

## Retrieve

1. **Check availability first.** `checkPatentDocumentAvailability`,
   `checkPatentAbstractAvailability`, `checkCaselawDocumentAvailability` return
   availability plus download URLs. Calling these before a bulk fetch avoids
   requesting documents your contract does not cover.
2. **Fetch documents.** `getPatentDocuments` (POST, many by GUID) or
   `getPatentDocument` (GET, one by GUID). Same pairs exist for trademarks
   (`getTrademarkDocuments` / `getTrademarkDocument`) and case law
   (`getCaselawDocuments` / `getCaselawDocument`).
3. **Fetch renditions.** `getPatentPdf`, `getPatentImages`,
   `derwentGetDocumentsPdf`, `derwentGetDocumentImages`. The PDF operations return
   **URLs**, not bytes — fetch the URL separately.
4. **Map identifiers across systems.** `getPublicationIdMapping` — POST
   `/patents/derwent/documents-IdMapping` — proxies the TSIP id-mapping service.
5. **Normalize entity names.** `getMatchPatentEntities` / `getMatchPatentEntity` run
   the entity-match model; use it before joining assignee names to your own CRM.

## Deprecated route

`POST /derwent-graphql` (the "DI GraphQL API") is marked **deprecated** on the
developer portal and can no longer be subscribed to. Use the REST operations above.

## Error handling

The gateway envelope applies: a rejected request returns
`{"message": "...", "request_id": "..."}` with 401. Backend errors return the
Clarivate `{"error": {...}}` shape. There is no RFC 9457 problem+json.

## Do not

- Do not treat a POST here as a write and do not retry-guard it with an idempotency
  key — Clarivate documents none, and these operations do not mutate state anyway.
- Do not assume `search-by-query` results are stable across calls; there is no cursor
  or snapshot semantic published.
