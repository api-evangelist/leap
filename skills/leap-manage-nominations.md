---
name: leap-manage-nominations
description: Offer meter capacity into Leap market programs — review Leap's nomination suggestions,
  accept or replace them per meter or in bulk, and search current nominations.
api: leap:nominations
apis:
  - leap:nominations
operations:
  - getMeterNominationSuggestions
  - postMeterNominationSuggestions
  - deleteMeterNominationSuggestion
  - postNominationSuggestions
  - searchNominationSuggestions
  - searchNominations
generated: '2026-07-19'
method: generated
source: openapi/leap-nominations-openapi-original.yml, https://developer.leap.energy/docs/bidding-introduction,
  https://developer.leap.energy/docs/api-responses
---

# Manage Leap nominations

A nomination is the capacity a meter is offered into a market program for a delivery period. It is
the commitment that dispatch is measured against and settlement pays on, so **every write in this
skill has revenue and penalty consequences** — require explicit human approval before posting or
deleting.

Base URL `https://api.leap.energy`, paths under `/v2/meters/…/nominations`, bearer key.
This is a 3.1.0 OpenAPI service.

## Read before you write

- `searchNominations` (`POST /v2/meters/nominations/search`) — the current nominations across the
  portfolio.
- `getMeterNominationSuggestions` (`GET /v2/meters/{meter_id}/nominations/suggestions`) — Leap's
  suggested nomination values for one meter.
- `searchNominationSuggestions` (`POST /v2/meters/nominations/suggestions/search`) — suggestions
  across many meters, for a bulk review pass.

## Write

- `postMeterNominationSuggestions` (`POST /v2/meters/{meter_id}/nominations/suggestions`) — set
  suggestions for a single meter.
- `postNominationSuggestions` (`POST /v2/meters/nominations/suggestions`) — set suggestions for many
  meters in one call.
- `deleteMeterNominationSuggestion`
  (`DELETE /v2/meters/{meter_id}/nominations/suggestions/{nomination_suggestion_id}`) — remove one.

**No idempotency key exists.** If a bulk post times out, do not blind-retry — re-read with
`searchNominationSuggestions` and reconcile first.

## Market validation codes

Bid and ask submissions are validated against market rules. A clean submission returns **HTTP 202**;
a failure returns a 4xx carrying one of Leap's numeric validation codes in the body. These codes look
like HTTP statuses but are not — see `errors/leap-error-codes.yml` for the full registry. The ones
that bite most often:

| Code | Meaning |
|---|---|
| 400 | Bid submission deadline for that trading day has passed |
| 401 | Bid price out of range — all prices must be under $1/kWh |
| 402 | Bid price scale invalid — maximum 5 decimals |
| 403 | Bid quantity out of range — must be greater than 1 kW |
| 404 | Timeslot falls in a closed market |
| 405 / 409 | Meters not assigned to a resource (at all, or on that trading day) |
| 406 / 407 | Timeslots must start on the hour and span exactly one hour |
| 408 / 415 | Meters or resources not assigned to this account |
| 410–412 | Bid curve below minimum quantity, price outside $0.01–$1.00, or not monotonically increasing |
| 413 / 414 | Over 100,000 bids per request, or over 100 meters per bid |
| 416–419 | Ask curve quantity non-positive, price out of range, non-increasing, or timeslot not ending on the hour |
| 430 | Duplicate bids for the same (meter id, bid start) |

Deadline (400) and duplicates (430) are the two you should guard for client-side before submitting.

## Pagination

`page_token` / `page_size` in the request body; loop on the returned next-page token.

## Transport errors

`400` invalid request, `401` unauthorized, `403` forbidden, `404` meter not found, `500` server
error, in the `{ title, status, details[] }` envelope.
