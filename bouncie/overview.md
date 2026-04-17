# Bouncie integration overview

Summarized from [Bouncie API docs](https://docs.bouncie.dev/#/) (OpenAPI 3.1, version `1.0.0` in the fetched spec).

## Two integration styles

1. **Webhooks (push)** — Bouncie POSTs JSON to your HTTPS endpoint when devices, trips, health, or geo-zone events occur. Configure URL and event types in the [Bouncie Developer Portal](https://www.bouncie.dev/).
2. **REST API (pull)** — Your backend calls `https://api.bouncie.dev/v1/…` with a **per-user access token** obtained via OAuth after a Bouncie customer authorizes your application.

Application registration (client ID/secret, redirect URIs, webhook URL and shared secret) is done in the Developer Portal.

## Environment URLs

| Purpose | URL |
|---------|-----|
| REST API | `https://api.bouncie.dev` — typical paths: `/v1/user`, `/v1/vehicles`, `/v1/trips`, etc. |
| OAuth authorization | `https://auth.bouncie.com/dialog/authorize` |
| OAuth token exchange / refresh | `https://auth.bouncie.com/oauth/token` |

All REST calls must use **HTTPS**; HTTP fails.

## Successful responses

- JSON bodies with `Content-Type: application/json` where applicable.
- Typical status codes: `200` (GET success), `201` (POST create), `204` (delete success with no body).

See [authentication.md](./authentication.md) for headers and [rest-api.md](./rest-api.md) for endpoint list.

## Geo-zones (high level)

Bouncie distinguishes **user** geo-zones (managed in the Bouncie app; integrations can read) from **application** geo-zones (created via API; not shown in Bouncie consumer UI). Workflow: create a **location** (GeoJSON polygon or point+circle), optionally a **schedule**, then create an **application geo-zone** referencing those IDs. Details: [geo-zones.md](./geo-zones.md).

## Further reading

- [Webhooks](./webhooks.md)
- [FAQ / operations](./faq-and-operations.md)
- Machine-readable spec: [openapi.json](./openapi.json)
