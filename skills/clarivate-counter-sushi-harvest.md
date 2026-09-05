---
name: clarivate-counter-sushi-harvest
description: >-
  Harvest COUNTER Release 5 usage statistics from Web of Science (or Cortellis Drug
  Discovery Intelligence) over the NISO SUSHI protocol. Use when a library or usage
  administrator needs Platform and Database reports pulled into an ERM or usage
  consolidation system.
api: Web of Science SUSHI COUNTER5 API
spec: openapi/clarivate-sushi-api-openapi.json
base_url: https://api.clarivate.com/api/counter/r5
operations:
  - getAPIStatus
  - getConsortiumMembers
  - getReports
  - getPRReport
  - getPRP1Report
  - getDRReport
  - getDRD1Report
  - getDRD2Report
generated: '2026-09-05'
method: generated
source: >-
  https://developer.clarivate.com/apis/sushi-api/swagger — operationIds above are
  declared verbatim in the harvested contract.
---

# Harvest COUNTER R5 usage over SUSHI

This is the one Clarivate API that implements an external standard end to end. If you
already have a COUNTER SUSHI client, point it at
`https://api.clarivate.com/api/counter/r5` and it will work — the paths are the
canonical COUNTER ones.

## Before you start

- This API is **approval-only, for site usage administrators**. Request it from an
  institutional email address; generic addresses are rejected. Approved Clarivate
  third-party partners may request it on behalf of a subscribing organization — name
  the third-party system in the request description.
- Authentication is **three-factor**: the API key (`X-ApiKey`), a **Requestor ID** and
  a **Customer ID**. All three are required on report calls, per the COUNTER R5 Code
  of Practice.

## Steps

1. **Check the service is reporting.** `getAPIStatus` — `GET /status`.
   A COUNTER 5.1 variant is served unauthenticated at
   `https://api.clarivate.com/api/counter/r51/status`; use it for a pre-flight check
   without spending a credentialed call.

2. **Resolve consortium members** if you harvest on behalf of several institutions:
   `getConsortiumMembers` — `GET /members`.

3. **List the reports the platform supports.** `getReports` — `GET /reports`.
   Clarivate exposes the Platform and Database families:
   - `getPRReport` — `GET /reports/pr` (Platform Master Report)
   - `getPRP1Report` — `GET /reports/pr_p1` (Platform Usage standard view)
   - `getDRReport` — `GET /reports/dr` (Database Master Report)
   - `getDRD1Report` — `GET /reports/dr_d1` (Database Search and Item Usage)
   - `getDRD2Report` — `GET /reports/dr_d2` (Database Access Denied)

4. **Pull a report** with `begin_date` and `end_date` in the COUNTER format the
   contract declares, plus your Requestor ID and Customer ID.

5. **Handle COUNTER exceptions.** COUNTER returns problems as `Exception` objects
   inside a 200 response as well as via HTTP status. Check the body, not only the
   status code.

## Cortellis

The same protocol is served for Cortellis Drug Discovery Intelligence at
`https://api.clarivate.com/api/cddi/counter5` with a smaller report set
(`/status`, `/reports`, `/reports/dr_d1`, `/reports/pr_p1`). See
`openapi/clarivate-cddi-counter-five-openapi.json`.

## Do not

- Do not build a bespoke parser. The payloads are COUNTER R5 — use a COUNTER library.
- Do not harvest more often than the Code of Practice expects; there is no documented
  throttle on this API and no rate-limit header to tell you when you are over.
