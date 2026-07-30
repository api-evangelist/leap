---
name: leap-subscribe-webhooks
description: Manage Leap webhook subscriptions on the general webhook platform — create, list, update,
  test and delete subscriptions for connect-session and meter/enrollment lifecycle events.
api: leap:webhooks
apis:
  - leap:webhooks
operations:
  - listWebhooks
  - createWebhook
  - updateWebhook
  - deleteWebhook
  - testWebhook
generated: '2026-07-19'
method: generated
source: openapi/leap-webhooks-openapi-original.yml, https://developer.leap.energy/docs/webhook-setup,
  https://developer.leap.energy/docs/retry-mechanism, https://developer.leap.energy/reference/metercreated
---

# Subscribe to Leap webhook events

The general webhook platform gives proactive visibility into portfolio changes so you can stop
polling meter inventory. Dispatch notifications are a **separate** surface — see
`leap-process-dispatch.md`.

Base URL `https://api.leap.energy` (staging `https://api.staging.leap.energy`), bearer key.

## Event types

Subscribe to one or more of these exact values:

| Event type | Fires when |
|---|---|
| `connect_session.updated` | A customer starts or progresses a Leap Connect session |
| `connect_session.authorization_updated` | The utility data-sharing authorization changes state |
| `meter.created` | A new meter is added to the partner account |
| `meter.enrollment.global-status.updated` | The meter's overall enrollment status changes |
| `meter.enrollment.participation-status.updated` | Participation preference (active/idle) changes |
| `meter.enrollment.required-actions.updated` | Leap needs the partner to do something |
| `meter.enrollment.group.updated` | VPP group membership changes |

Every payload carries a `webhook_event_type` discriminator field naming one of the above; branch on
it rather than on payload shape.

## Steps

1. **Stand up a receiver.** HTTPS on port 443, 8443 or 8843, TLS 1.2 or 1.3, returning any `2xx`
   within 10 seconds. For a throwaway receiver during development Leap's docs point at
   https://webhook.site.
2. **List what exists.** `listWebhooks` (`GET /v1.1/webhooks`) — never create a duplicate subscription
   for a URL that is already registered.
3. **Create.** `createWebhook` (`POST /v1.1/webhooks`) with the receiver `url`, the event types, and
   any custom `headers` (name/value) Leap should send — use one for a shared secret so your receiver
   can verify the caller.
4. **Test.** `testWebhook` (`POST /v1.1/webhooks/{webhook_id}/test`) sends a test event to the
   configured `receiver_url`. Do this before relying on the subscription.
5. **Update carefully.** `updateWebhook` (`PUT /v1.1/webhooks/{webhook_id}`) **overwrites all existing
   settings** for that webhook_id — it is not a patch. Read the current record with `listWebhooks`,
   merge your change client-side, then send the complete object.
6. **Delete.** `deleteWebhook` (`DELETE /v1.1/webhooks/{webhook_id}`) stops delivery immediately.

## Delivery semantics

- Up to 10 attempts over ~8 hours: immediate, 1m, 2m, 4m, 8m, 16m, 29m, 1h, 2h, 4h. After the 10th
  failure Leap stops trying.
- Calls time out at 10 seconds. Acknowledge fast and process asynchronously.
- Because retries exist and Leap publishes no delivery-id dedupe guarantee, **make your handler
  idempotent on your side** — key on `meter_id` plus the event type plus the state being reported.

## Errors

The webhook service uses its own envelope — `{ error, message }`. Expect `401` (auth token invalid),
`403` (no permission), `404` (webhook_id not found) and `500`.

## Alternative

Webhook subscriptions can also be managed from the Partner Portal —
production `https://partner.leap.energy/account?settings=webhooks`,
staging `https://partner.staging.leap.energy/account?settings=webhooks`.
