---
name: leap-process-dispatch
description: Receive and act on Leap grid dispatch instructions — register meter-level and group-level
  dispatch webhooks, run integration and communication tests, and poll dispatch events as a fallback.
api: leap:dispatch
apis:
  - leap:dispatch
operations:
  - getMeterDispatchWebhook
  - createOrUpdateMeterDispatchWebhook
  - deleteMeterDispatchWebhook
  - triggerTestMeterDispatchNotification
  - getGroupDispatchWebhook
  - createOrUpdateGroupDispatchWebhook
  - deleteGroupDispatchWebhook
  - triggerTestGroupDispatchNotification
  - triggerCommunicationTestMeterDispatch
  - searchMeterDispatches
  - searchGroupDispatches
generated: '2026-07-19'
method: generated
source: openapi/leap-dispatching-openapi-original.yml, https://developer.leap.energy/docs/dispatch-automation-v2,
  https://developer.leap.energy/docs/dispatch-event-processing, https://developer.leap.energy/docs/webhook-push-notifications-v2,
  https://developer.leap.energy/docs/dispatch-event-polling-v2
---

# Process Leap dispatch events

Dispatch is the revenue-bearing path: Leap tells you when and how much load to shift, your devices
respond, and settlement pays on measured performance. Getting this wrong costs the partner money, so
treat every change here as a production change.

Base URL is `https://api.leap.energy/v2` (staging `https://api.staging.leap.energy/v2`), bearer key.

## Choose push or poll

**Push (recommended).** Leap posts dispatch notifications to a URL you register. There is one
meter-level URL and one group-level URL per account — these are *not* the general webhook platform
subscriptions, they are separate registrations on the dispatch service.

**Poll (fallback).** Search for dispatch events on a schedule.

## Registering dispatch webhooks

1. `getMeterDispatchWebhook` (`GET /dispatch/meter/webhook`) — read what is registered today. Do this
   before changing anything; the operation overwrites, it does not merge.
2. `createOrUpdateMeterDispatchWebhook` (`PUT /dispatch/meter/webhook`) — set the `url` plus any
   `headers` (name/value pairs) Leap should send, e.g. a shared secret.
3. `triggerTestMeterDispatchNotification` (`POST /dispatch/meter/webhook/integration_test`) — send a
   synthetic dispatch to your receiver. The response returns `http_status`, `sent_time`,
   `received_time`, `headers`, `body` and `error_message`, so you can debug without waiting for a real
   grid event.
4. Repeat with `getGroupDispatchWebhook`, `createOrUpdateGroupDispatchWebhook` and
   `triggerTestGroupDispatchNotification` (`/dispatch/group/webhook*`) if the partner dispatches at
   market-group level.
5. `deleteMeterDispatchWebhook` / `deleteGroupDispatchWebhook` (`DELETE`) remove a registration — this
   stops live dispatch delivery, so confirm with a human first.

### Receiver requirements

HTTPS on port **443, 8443 or 8843**, TLS **1.2 or 1.3**. Return any `2xx` within **10 seconds** — a
`2xx` at 11 seconds is still counted as a failure. Leap retries up to 10 times over roughly 8 hours
(immediate, 1m, 2m, 4m, 8m, 16m, 29m, 1h, 2h, 4h) and then stops.

## Reading dispatch payloads

Meter-level payloads carry `meter_id`, `partner_reference` and `timeslots[]`. Group-level payloads
carry `market_group_id`, `meter_ids[]`, `partner_references[]` and `timeslots[]`. Each timeslot has:

- `start_time`, `end_time`
- `energy_kw` — the dispatch target
- `nomination_kw` — what was offered into the market
- `cancelled` — **check this**; a later notification can cancel an earlier timeslot
- `priority`, `is_voluntary`, `dispatch_event_types`, `programs`
- `performance_compensation_cap`

Treat `is_voluntary` and `cancelled` as first-class: acting on a cancelled timeslot dispatches
customer assets for nothing, and treating a voluntary event as mandatory burns customer goodwill.

## Polling instead

- `searchMeterDispatches` (`POST /dispatch/meter/search`)
- `searchGroupDispatches` (`POST /dispatch/group/search`)

Both take `start_date`, `end_date`, `exclude_real_time_events`, `exclude_voluntary_events` and
`page_token`, and return `next_page_token` with `results[]`. Loop on `next_page_token`.

## Verifying end-to-end control

`triggerCommunicationTestMeterDispatch` (`POST /dispatch/meter/communication_test`) creates a test
dispatch against real meters — `start_time`, `end_time`, `test_event`, `program`, `meter_uuids`,
`dispatch_quantity_kw`, `approved`. **This moves real customer assets.** Never call it without
explicit human approval, and prefer the staging environment.

## Errors

`400` unparseable request, `401` bad token, `403` no access to this resource, `404` resource not
found, and a wildcard `5XX` unexpected error. The dispatch service uses its own envelope —
`{ code, message, error_code }` — not the `{ title, status, details[] }` shape the other Leap
services return. Branch accordingly.

## Test cases

Leap publishes a dispatch event-processing test suite at
https://developer.leap.energy/docs/event-processing-test-cases — run your handler against it before
going live.
