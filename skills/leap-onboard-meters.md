---
name: leap-onboard-meters
description: Onboard customer meters into a Leap virtual power plant — create meters in bulk or singly,
  poll the batch job to completion, and confirm each meter's details and enrollment status.
api: leap:create-meters
apis:
  - leap:create-meters
  - leap:meters
  - leap:enrollments
operations:
  - createMeterBatchJob
  - getMeterBatchJob
  - listMeterBatchJobs
  - listMeterBatchJobsFilters
  - createSingleMeter
  - listProvisionalAssets
  - getMeterDetails
  - searchMeterDetails
  - getMeterEnrollment
  - searchMeterEnrollments
generated: '2026-07-19'
method: generated
source: openapi/leap-create-meters-openapi-original.yml, openapi/leap-meters-openapi-original.yml, openapi/leap-enrollments-openapi-original.yml,
  https://developer.leap.energy/docs/partner-created-meters, https://developer.leap.energy/docs/onboard-overview
---

# Onboard meters into Leap

Use this skill to add a partner's customer meters to the Leap platform and confirm they reached an
enrolled state. A meter is the unit of inventory that provides grid services — the utility service
point plus the devices behind it.

## Before you start

- Authorize every call with `Authorization: Bearer <API_KEY>`. Keys are issued per environment in the
  Leap Partner Portal and a staging key against production (or the reverse) returns **403**.
- Production is `https://api.leap.energy`; staging is `https://api.staging.leap.energy`. Do dry runs
  in staging using the CAISO/NYISO staging demo utilities (see `sandbox/leap-sandbox.yml`).
- **Leap has no idempotency key.** A retried create is a second create. Always set `partner_reference`
  on each meter so you can detect duplicates with `searchMeterDetails` before retrying.

## Steps

1. **Decide bulk or single.** For more than a handful of meters call `createMeterBatchJob`
   (`POST /v1.1/jobs/meters`); it accepts CSV or JSON and returns a job ID. For one meter call
   `createSingleMeter` (`POST /v1.1/meters`).
2. **Check the available filters first** if you need to reconcile against existing uploads —
   `listMeterBatchJobsFilters` (`GET /v1.1/jobs/meter-job-filters`) returns the valid filter values,
   and `listMeterBatchJobs` (`GET /v1.1/jobs/meters`) lists prior jobs and their status.
3. **Poll the job.** Call `getMeterBatchJob` (`GET /v1.1/jobs/meters/{job_id}`) until it reports a
   terminal status. The response carries the meter IDs for the rows that uploaded successfully.
4. **Read the row-level failures.** A `400` on `createMeterBatchJob` returns `errors[]` with
   `{ field, description, suggestions }` and validation entries carrying `csv_row` or `json_index` and
   `problem_field`. Fix those rows and resubmit only the failures — never the whole file.
5. **Check for provisional assets.** `listProvisionalAssets` (`GET /v1.1/provisional-assets`) shows
   meter candidates still awaiting utility data or eligibility resolution.
6. **Confirm the meter.** `getMeterDetails` (`GET /v2/meters/{meter_id}`) returns customer, site,
   device and utility detail. To reconcile a whole batch, call `searchMeterDetails`
   (`POST /v2/meters/search`) filtering on `partner_references`.
7. **Confirm enrollment.** `getMeterEnrollment` (`GET /v2/meters/{meter_id}/enrollments`) returns the
   global enrollment status, participation preferences, program enrollment and — critically —
   `required_actions`. Sweep the portfolio with `searchMeterEnrollments`
   (`POST /v2/meters/enrollments/search`) using `partner_action_required: true` to find the meters
   that are blocked on you.

## Pagination

Both search endpoints use body-level `page_token` / `page_size` and return `next_page_token`. Loop
until `next_page_token` is absent or empty. Do not assume a page count.

## Errors to expect

| Status | Meaning | Do this |
|---|---|---|
| 400 | Invalid filter, body, or row | Read `details[]` / `errors[]` and fix the named field |
| 401 | Missing or malformed Authorization header | Send `Bearer <API_KEY>` |
| 403 | Wrong environment key, or key lacks the permission | Check the key on the Partner Account page |
| 404 | Meter not found for this account | Verify `meter_id`; it may belong to another tenant |
| 500 | Server error | Retry with backoff, then check https://status.leap.energy/ |

Error bodies on these services are `{ title, status, details[] }` over `application/json` — not
RFC 9457 problem details.

## Prefer events over polling

Rather than re-polling meter inventory, subscribe to `meter.created` and the
`meter.enrollment.*` webhook events — see `leap-subscribe-webhooks.md` and
`asyncapi/leap-events-asyncapi.yml`. Leap also announced that Meters API **v1**
(`getMeter`, `searchMeters`) sunsets **2026-10-31**; build only on the v2 operations above.
