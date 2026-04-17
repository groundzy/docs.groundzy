# Bouncie FAQ and operational notes

Consolidated from the [Bouncie API documentation](https://docs.bouncie.dev/#/) FAQ and related sections.

## Why does the API return 401 Unauthorized?

1. **Expired access token** — refresh using `grant_type=refresh_token` at `https://auth.bouncie.com/oauth/token` ([authentication.md](./authentication.md)).
2. **Wrong `Authorization` format** — send **only** the access token string for REST calls, **not** `Bearer <token>`.
3. **Access revoked or invalid user linkage** — confirm the user appears under **Users & Devices** for your app in the [Developer Portal](https://www.bouncie.dev/).

## How much webhook data will I receive?

Depends on:

- Number of authorized devices and how often vehicles move.
- Which event types are enabled.

Examples from the docs:

- **`battery`** — only when voltage drops below a threshold.
- **`tripMetrics`** — about once per trip when the trip completes.
- **`tripData`** — continuous during trips; dominates volume if enabled.

Connectivity gaps (garages, weak cell) buffer events on the device and can produce **short bursts** when back online — total data volume is unchanged but arrival rate spikes.

## Why duplicate trip events?

See [webhooks.md](./webhooks.md): parallel **real-time** and **periodic** streams cause overlapping payloads. Use **`transactionId`** and timestamps to dedupe and reconcile.

## What if webhook delivery fails?

- Retries with **exponential backoff** until a maximum attempt count.
- Persistent failure can **deactivate** the webhook URL until you fix the endpoint and **re-enable** it in the portal.

## Zapier integration

Bouncie documents a dedicated Zapier OAuth redirect:

`https://zapier.com/dashboard/auth/oauth/return/App220583CLIAPI/`

Register a **separate** OAuth application used **only** for Zapier if you use this path — do not mix with your production Groundzy client credentials.

## Related local docs

- [authentication.md](./authentication.md)
- [webhooks.md](./webhooks.md)
- [rest-api.md](./rest-api.md)
