# Leap

Leap is a San Francisco-based energy software company that lets technology brands build and scale
virtual power plants (VPPs). Its platform aggregates distributed energy resources — residential and
commercial battery storage, smart thermostats and heat pumps, and EV charging — and gives them a single
point of integration into wholesale electricity markets and utility grid-service programs across CAISO,
NYISO and other regions.

Website: https://www.leap.energy/ · Developer portal: https://developer.leap.energy/ ·
Status: https://status.leap.energy/ · Backed by: Union Square Ventures

## Identity note

This repo was created by the VC-portfolio pipeline as a stub pointing at `leapwallet.io` (Leap Wallet,
a Cosmos crypto wallet backed by CoinFund and Pantera that announced shutdown effective 2026-05-28),
with `accel`, `creandum` and `union-square-ventures` listed as backers. The Union Square Ventures
portfolio company named Leap is the energy/VPP platform at `leap.energy` (Series A led by USV, January
2020), which is the company profiled here. The Accel and Creandum attributions could not be verified
against either company and were removed.

## APIs

Seven versioned services on one host pair (`https://api.leap.energy`, staging
`https://api.staging.leap.energy`), all authorized with a portal-issued bearer API key. 42 operations
across the published OpenAPI definitions.

| API | Spec |
|---|---|
| Meter Details | `openapi/leap-meters-openapi-original.yml` |
| Meter Enrollment | `openapi/leap-enrollments-openapi-original.yml` |
| Create Meters (bulk) | `openapi/leap-create-meters-openapi-original.yml` |
| Dispatch v2 | `openapi/leap-dispatching-openapi-original.yml` |
| Meter Nominations | `openapi/leap-nominations-openapi-original.yml` |
| Revenue & Analytics | `openapi/leap-settlement-openapi-original.yml` |
| Webhook Subscriptions | `openapi/leap-webhooks-openapi-original.yml` |

## Artifacts

`asyncapi/` · `authentication/` · `changelog/` · `components/` · `conformance/` · `conventions/` ·
`data-model/` · `errors/` · `lifecycle/` · `llms/` · `mcp/` · `openapi/` · `overlays/` · `packages/` ·
`sandbox/` · `security/` · `skills/` · `well-known/`

Notable findings: Leap publishes a real `llms.txt`, a dated changelog with a named sunset date for
Meters API v1 (2026-10-31), a full staging environment with synthetic CAISO/NYISO demo utilities, and
a seven-event webhook catalog. It publishes no SDKs, no GitHub organization, no `/.well-known/`
documents, no idempotency mechanism, no rate-limit signalling, and no compliance certifications.
