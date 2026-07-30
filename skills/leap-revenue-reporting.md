---
name: leap-revenue-reporting
description: Pull Leap settlement revenue — monthly and annual reports, revenue aggregated by meter,
  customer, load type, market group or utility, and the monthly unresponsive-meter diagnostic that
  explains missed revenue.
api: leap:revenue
apis:
  - leap:revenue
operations:
  - getPeriodicReports
  - getPeriodicReportVersions
  - getYearlyOverview
  - postMeterSearchPeriodicAggregationSearch
  - postCustomerSearchPeriodicAggregationSearch
  - postLoadTypeSearchPeriodicAggregationSearch
  - postMarketGroupSearchPeriodicAggregationSearch
  - postUtilitySearchPeriodicAggregationSearch
  - postMonthlyUnresponsiveMetersPerformance
generated: '2026-07-19'
method: generated
source: openapi/leap-settlement-openapi-original.yml, https://developer.leap.energy/docs/revenue-settlement-data,
  https://developer.leap.energy/changelog/unresponsive-meters-api-released
---

# Report on Leap revenue and performance

All operations are on `https://api.leap.energy` under `/v1.1/revenue` and
`/v1.1/performance`, with a bearer key. Every endpoint is read-only — this skill is safe to run
without human approval.

## Versioning is the thing to get right

Settlement figures are **restated**. `getPeriodicReports`
(`GET /v1.1/revenue/periodic/reports`) returns the latest version when no version is specified;
`getPeriodicReportVersions` (`GET /v1.1/revenue/periodic/versions`) lists every version for a period.
If you are reconciling against a number a partner saw last month, fetch the version they saw — do not
assume the latest matches. The aggregation search requests accept a `version` field for this.

Report status is carried on `report_status` / `SettlementReportStatus`; treat preliminary figures as
provisional (`YearlySummary` distinguishes `actual_revenue` from `preliminary_revenue`).

## The five aggregations

All are `POST /v1.1/revenue/periodic/aggregation/{dimension}/search` and share a request shape:
`period_start`, `period_end`, `programs`, `filters`, `page_token`, `page_size`, `sort` (with `field`
and `direction`), and `version`.

| Operation | Groups revenue by |
|---|---|
| `postMeterSearchPeriodicAggregationSearch` | meter (`meter_id`, `partner_reference`) |
| `postCustomerSearchPeriodicAggregationSearch` | customer name / customer group |
| `postLoadTypeSearchPeriodicAggregationSearch` | load type |
| `postMarketGroupSearchPeriodicAggregationSearch` | market group |
| `postUtilitySearchPeriodicAggregationSearch` | utility |

Common measures across the aggregations: `nomination_w`, `monetized_w`, `capacity_earned_w`,
`energy_earned_wh`, `capacity_revenue`, `energy_revenue`, `total_revenue`, `potential_revenue`,
`capacity_performance_percentage`, `energy_performance_percentage` and `data_coverage_percentage`.

**Read `data_coverage_percentage` before you quote any number.** Leap gap-fills missing utility
interval data by prediction; low coverage means the revenue figure rests on estimated intervals.

The gap between `total_revenue` and `potential_revenue` is the story worth telling — that is money
the portfolio left on the table.

## Annual view

`getYearlyOverview` (`GET /v1.1/revenue/yearly/{year}`) returns `yearly_summary` (including
`missed_revenue`, `payment_adjustment`, `penalty`, `performance_adjustment` and month-by-month
revenue), `revenue_sources` split into capacity and energy, `regional_economics` by transmission
region, `unit_economics` (average revenue per meter, per market group, per kW) and the underlying
`reports`.

## Explaining the gap

`postMonthlyUnresponsiveMetersPerformance`
(`POST /v1.1/performance/diagnosis/monthly/unresponsive/meters`) is the diagnostic that turns missed
revenue into a work list. Request by `year_month` with optional `meter_id_contains`,
`partner_reference_contains`, `customer_name_contains`, `transmission_regions`, `programs`,
`historical_responsiveness`, `include_meter_programs` / `exclude_meter_programs`, plus paging and
sort. The response gives a `summary` (`number_of_meters`, `missed_revenue`,
`average_response_percentage`) and per-meter rows with `response_percentage`, `number_of_events`,
`missed_revenue` and `historical_responsiveness`.

This endpoint left beta recently — it is the intended way to automate alerting on non-responsive
meters.

## Pagination

Body-level `page_token` / `page_size`; responses return `page_token` for the next page. Loop until
it is absent.

## Errors

`400` invalid request, `401` unauthorized, `403` forbidden, **`415` unsupported media type** (send
`application/json` — this service is stricter than the others), `500` server error. Envelope is
`{ title, status, details[] }` with `details[]` entries of `{ message, description }`.
