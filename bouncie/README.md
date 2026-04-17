# Bouncie API — Groundzy reference

This folder holds **local reference material** for integrating [Bouncie](https://www.bouncie.com/) fleet GPS hardware and telematics with Groundzy (teams, maps, workflows). Use it alongside the live specification.

## Official sources

| Resource | URL |
|----------|-----|
| Interactive API docs (OpenAPI UI) | [https://docs.bouncie.dev/](https://docs.bouncie.dev/) |
| OpenAPI 3.1 JSON (same spec as the UI) | [https://docs.bouncie.dev/openapi.json](https://docs.bouncie.dev/openapi.json) |
| Developer Portal (app registration, OAuth client, webhooks) | [https://www.bouncie.dev/](https://www.bouncie.dev/) |
| REST base URL | `https://api.bouncie.dev` (paths are under `/v1/…`) |
| OAuth authorize | `https://auth.bouncie.com/dialog/authorize` |
| OAuth token | `https://auth.bouncie.com/oauth/token` |

## Files in this folder

| File | Contents |
|------|----------|
| [overview.md](./overview.md) | Integration model (REST vs webhooks), base URLs, HTTP behavior |
| [authentication.md](./authentication.md) | OAuth 2.0 authorization code flow, PKCE, token refresh, API headers |
| [rest-api.md](./rest-api.md) | REST endpoints summary (user, vehicles, trips, locations, schedules, geo-zones, webhooks CRUD) |
| [webhooks.md](./webhooks.md) | Inbound webhook security, retries, event types, duplication and disconnect behavior |
| [geo-zones.md](./geo-zones.md) | User vs application geo-zones; creating zones via API |
| [faq-and-operations.md](./faq-and-operations.md) | FAQ from official docs (401s, data volume, duplicates, delivery failures), Zapier note |
| [groundzy-teams-fleet.md](./groundzy-teams-fleet.md) | How this ties to Groundzy **teams** / fleet tracking (design considerations, not product decisions) |
| [openapi.json](./openapi.json) | Frozen copy of the official OpenAPI document (for diffing and offline review; refresh periodically) |

## Refreshing `openapi.json`

When Bouncie ships API changes, replace the local file:

```powershell
Invoke-WebRequest -Uri "https://docs.bouncie.dev/openapi.json" -OutFile "openapi.json"
```

Then diff `rest-api.md` / this README if paths or behavior changed.
