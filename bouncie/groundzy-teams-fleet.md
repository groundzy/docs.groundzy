# Groundzy teams × Bouncie fleet tracking — integration notes

This document connects Bouncie’s **per–Bouncie-account OAuth model** to Groundzy **teams** and fleet-style features. It is **engineering context**, not a finalized product spec.

## What Bouncie gives you

- **Devices** identified by **IMEI** tied to a **Bouncie user account** that authorized your app.
- **Vehicles** searchable by VIN/IMEI; **trips** with optional GeoJSON/polyline paths; **locations/schedules/geo-zones** for rules; **webhooks** for live telemetry and events.

Official references: [overview.md](./overview.md), [rest-api.md](./rest-api.md), [webhooks.md](./webhooks.md).

## Authorization reality (important for teams)

OAuth tokens are issued in the context of **one Bouncie login**. Data visible through `/v1/vehicles`, `/v1/trips`, etc. is scoped to **that** authorization.

For a Groundzy **team** that operates multiple trucks, you must decide explicitly:

| Approach | Idea | Tradeoffs |
|----------|------|-----------|
| **Single fleet account** | One org-owned Bouncie account; all devices live under it | One OAuth connection per team/org; simplest token management; drivers use Bouncie app under that account if needed |
| **Multi-account** | Each driver (or subcontractor) has their own Bouncie account | Multiple OAuth tokens to store and refresh; clearer legal ownership per driver; more consent flows |
| **Hybrid** | Org fleet + occasional personal devices | Most flexible; most complex mapping and privacy rules |

Groundzy should store **tokens and refresh tokens** server-side (encrypted), tied to **`teamId` / `organizationId`** and metadata about **which Bouncie identity** they represent.

## Suggested backend surfaces (conceptual)

- **OAuth callback route** — exchange `code`, persist refresh token, associate with team + acting user.
- **Token refresh job** — proactive refresh before expiry; alert on revocation.
- **Webhook ingress** — verify `Authorization` / `X-Bouncie-Authorization`; map IMEI/VIN → internal vehicle/team IDs; enqueue for trip processing.
- **Sync jobs** — optional backfill via `GET /v1/trips` and `/v1/vehicles` when webhooks were down.

## Mapping to Groundzy concepts

Existing docs (`groundzy-v3-docs/06-features/teams-and-sharing.md`) describe teams, org membership, and roles. Fleet alignment might introduce:

- **Fleet vehicle** records (internal id, label, **IMEI**, optional VIN, linked **teamId**, optional link to workflow assets).
- **Telemetry** events stored under team-scoped collections or BigQuery-style logs for history.
- **Application geo-zones** ([geo-zones.md](./geo-zones.md)) for job sites, yards, or dispatch corridors — independent of end-user zones in the Bouncie consumer app.

Exact Firestore shapes and permissions belong in a dedicated tech design when you implement.

## Compliance and UX

- Disclose what location/trip data you process and retention policy.
- Allow disconnect/revoke (token delete + portal “remove app” path).
- Rate-limit webhook handlers; dedupe trips using **`transactionId`** ([webhooks.md](./webhooks.md)).

## Next steps when implementing

1. Register the app in the [Developer Portal](https://www.bouncie.dev/) and choose redirect URIs for Groundzy environments.
2. Confirm fleet model (single vs multi Bouncie account) with stakeholders.
3. Implement OAuth + webhook verification first; add REST polling as backup for gaps.
4. Refresh [openapi.json](./openapi.json) periodically and diff behavior.
