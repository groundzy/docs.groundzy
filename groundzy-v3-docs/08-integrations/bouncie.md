# Bouncie

## What it does

**Fleet GPS and telematics:** OAuth-connected access to vehicles, trips, locations, schedules, application geo-zones, and optional **webhooks** for live device/trip/geo events. Intended for **teams** fleet tracking and management alongside Groundzy org membership.

## Status

**Planning / integration design** — no production wiring in app until OAuth, webhook ingress, and team-scoped token storage are implemented.

## Full reference

Authoritative technical pack (URLs, OAuth, REST table, webhooks, geo-zones, FAQ, frozen OpenAPI):

**[`../../bouncie/README.md`](../../bouncie/README.md)**

Start there; nested files cover authentication, REST paths, webhooks, and **Groundzy teams × fleet** modeling notes.

## How it’s used (target shape)

| Area | Connection |
|------|------------|
| **OAuth** | `https://auth.bouncie.com` — per Bouncie account; tokens stored server-side per team/org decision |
| **REST** | `https://api.bouncie.dev/v1/*` — vehicles, trips, locations, geo-zones (see reference pack) |
| **Webhooks** | HTTPS endpoint — verify `Authorization` / `X-Bouncie-Authorization`; map IMEI/VIN to internal fleet entities |

## Risks & constraints

| Risk | Mitigation / note |
|------|-------------------|
| **Per-account OAuth scope** | Fleet model must be explicit (single org Bouncie account vs many drivers); see [`../../bouncie/groundzy-teams-fleet.md`](../../bouncie/groundzy-teams-fleet.md) |
| **Webhook secret** | Validate every payload; support rotation via response header |
| **Trip duplication** | Dedupe with `transactionId` and timestamps per Bouncie docs |
| **Token handling** | Refresh before expiry; no client-side long-lived secrets |

## Related

- [`../../bouncie/groundzy-teams-fleet.md`](../../bouncie/groundzy-teams-fleet.md) — teams alignment
- [`../06-features/teams-and-sharing.md`](../06-features/teams-and-sharing.md) — teams data model
