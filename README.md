# Leap

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
