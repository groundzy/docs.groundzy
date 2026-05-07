# Bouncie

## What it does

**Fleet GPS and telematics:** OAuth-connected access to Bouncie vehicles and optional **webhooks** for live trip/location events. **Teams** tier orgs connect one Bouncie account per Groundzy team; tokens stay server-side; the map can show live vehicle positions when the **Fleet** map filter is on.

## Status

**Implemented in app** (OAuth, webhooks, sync, map layer, and per-vehicle display settings).

## Full reference

Authoritative technical pack (URLs, OAuth, REST table, webhooks, geo-zones, FAQ, frozen OpenAPI):

**[`../../bouncie/README.md`](../../bouncie/README.md)**

## How it works in Groundzy

| Area | Behavior |
|------|----------|
| **OAuth** | `https://auth.bouncie.com` — PKCE; refresh/access tokens in `teams/{teamId}/bouncie_private/auth` (client cannot read). |
| **Post-connect sync** | After a successful token exchange, the server runs a **vehicle sync** (with timeout). This writes `bouncie_account_routing/imei:{imei}` for every device IMEI returned by the API, even if the vehicle has no GPS in the vehicles payload yet — so **`tripData`** webhooks can route before the first coordinates appear in `/v1/vehicles`. |
| **REST** | `https://api.bouncie.dev/v1/*` — e.g. paginated `GET /vehicles` for poll + IMEI routing hints. |
| **Webhooks** | HTTPS endpoint validates shared secret (`Authorization` / `X-Bouncie-Authorization`). Payloads are normalized to update `teams/{teamId}/bouncie_vehicle_locations` when coordinates and org routing resolve (Bouncie user id and/or `imei:` routing docs). |
| **Map** | Client reads `bouncie_vehicle_locations` and optional `bouncie_fleet_vehicle_display` for label/color/icon. |
| **Fleet settings UI** | Owners/admins can set **nickname**, **marker color**, **map icon preset**, and **photo** (Storage) per vehicle without overwriting telemetry written by the server. |

## Data locations (Firestore)

- `teams/{teamId}.bouncieFleet` — connection metadata (no secrets).
- `teams/{teamId}/bouncie_private/` — OAuth tokens (admin-only).
- `teams/{teamId}/bouncie_vehicle_locations/{docId}` — latest lat/lng and device hints (admin-written; members read).
- `teams/{teamId}/bouncie_fleet_vehicle_display/{vehicleDocId}` — user-edited display fields (keyed by location doc id).
- `bouncie_account_routing/{accountId}` — routes Bouncie user id or `imei:{imei}` → `organizationId` (admin-only).

## Risks & constraints

| Risk | Mitigation / note |
|------|------------------|
| **Per-account OAuth scope** | One Bouncie account per connected team; see [`../../bouncie/groundzy-teams-fleet.md`](../../bouncie/groundzy-teams-fleet.md) |
| **Webhook secret** | Compare incoming `Authorization` / `X-Bouncie-Authorization` to configured secret |
| **Trip duplication** | Webhook path dedupes with content hash under `bouncie_webhook_dedupe` |
| **Token handling** | Refresh before expiry; nothing long-lived in the client |
| **Sync timeouts** | Post-OAuth sync errors surface as `bouncieFleet.lastFleetSyncError`; user can run **Sync vehicles** again |

## Related

- [`../../bouncie/groundzy-teams-fleet.md`](../../bouncie/groundzy-teams-fleet.md) — teams alignment
- [`../06-features/teams-and-sharing.md`](../06-features/teams-and-sharing.md) — teams data model
