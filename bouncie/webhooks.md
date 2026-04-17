# Bouncie webhooks

Sources: [Bouncie API — Webhooks](https://docs.bouncie.dev/#/), OpenAPI `webhooks` section in [openapi.json](./openapi.json).

## Purpose

Instead of polling REST endpoints, configure an HTTPS endpoint in the [Bouncie Developer Portal](https://www.bouncie.dev/). Bouncie POSTs JSON payloads when subscribed events occur for authorized vehicles/users.

## Security

Each delivery includes headers you configure at registration time:

```http
Authorization: <your-shared-secret>
X-Bouncie-Authorization: <same-value>
```

`X-Bouncie-Authorization` duplicates `Authorization` for platforms that strip the standard header.

Validate incoming requests against your stored secret before processing.

### Rotating the webhook secret

Respond to a webhook with a new secret in an **`Authorization` response header**; Bouncie updates the key for subsequent deliveries.

## Delivery and retries

- Respond with **`2xx`** to acknowledge receipt.
- Retries occur on timeout, invalid JSON response, or **`4xx` / `5xx`** responses, using **backoff**, up to a **maximum** retry count.

If delivery keeps failing, the webhook URL may be **automatically deactivated** until re-enabled in the portal — then no events are delivered.

## Event types (outbound payloads)

OpenAPI webhook operation names map to portal event toggles:

| Category | Events |
|----------|--------|
| Device | Device Connect, Device Disconnect, VIN Change |
| Vehicle health | MIL (malfunction indicator), Low Battery |
| Trips | Trip Start, Trip Data, Trip Metrics, Trip End |
| Geo-zones | Application Geo-Zone, User Geo-Zone |

## Official guidance: duplication (trips)

Trip-related streams may duplicate logical events. From the spec:

- **Real-Time Stream** — fast delivery; may overlap with periodic data.
- **Periodic Stream** — ordered/interval batches; overlaps with real-time.

**Mitigation:**

1. Deduplicate using **`transactionId`** per trip where present.
2. Use **timestamps** to judge freshness and reconcile periodic vs real-time points.

Trip Start / Trip Data / Trip Metrics / Trip End descriptions in OpenAPI repeat this guidance.

## Official guidance: device disconnect

Disconnect events fire when external power is lost (unplug/tamper).

You may see **two** disconnect notifications for one physical incident:

1. **Immediate** — sent when possible if connectivity allows.
2. **Deferred** — recorded on device and sent after power returns and modem reconnects.

Treat multiple disconnects as **one logical event** where appropriate.

## Managing webhook registrations via REST

See [rest-api.md](./rest-api.md) — `GET/POST /v1/webhooks`, `PUT/DELETE /v1/webhooks/{webhookId}` — for programmatic create/update/delete of webhook configs (still requires OAuth as for other `/v1` routes).

## Data volume expectations

Brief notes from FAQ (full text: [faq-and-operations.md](./faq-and-operations.md)):

- **`battery`** — only when voltage crosses threshold.
- **`tripMetrics`** — roughly once per trip at end.
- **`tripData`** — continuous during trips; highest volume if enabled.

Cellular outages can cause **burst** delivery when the device catches up — expect short traffic spikes without a net increase in total trip data.
