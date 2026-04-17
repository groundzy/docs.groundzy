# Bouncie REST API endpoints

Base URL from spec: **`https://api.bouncie.dev`** — all documented paths below are prefixed with **`/v1`** (full example: `GET https://api.bouncie.dev/v1/vehicles`).

Authentication: send the user **access token** in the **`Authorization`** header (**no** `Bearer ` prefix per [FAQ](./faq-and-operations.md)).

The tables below summarize operations in the frozen [openapi.json](./openapi.json). Request/response JSON shapes are fully specified inline in that file (schemas are not split into `#/components/schemas`).

## User

| Method | Path | Summary |
|--------|------|---------|
| GET | `/v1/user` | Get User |

## Vehicles

| Method | Path | Summary |
|--------|------|---------|
| GET | `/v1/vehicles` | Search Vehicles |

Query parameters (all optional unless noted):

| Parameter | Description |
|-----------|--------------|
| `vin` | Vehicle identification number |
| `imei` | Bouncie device identifier (15-digit style per other endpoints) |
| `limit` | Page size (`number`, \> 0) |
| `skip` | Offset for paging (`number`, \> 0) |

## Trips

| Method | Path | Summary |
|--------|------|---------|
| GET | `/v1/trips` | Search Trips |

Query parameters:

| Parameter | Description |
|-----------|--------------|
| `imei` | Vehicle/device to filter trips |
| `transaction-id` | Unique trip identifier |
| `gps-format` | `geojson` or `polyline` |
| `starts-after` | ISO date-time — window with `ends-before` **must not exceed one week**; default last week if omitted |
| `ends-before` | ISO date-time |

## Locations

| Method | Path | Summary |
|--------|------|---------|
| GET | `/v1/locations/` | Get Locations |
| POST | `/v1/locations/` | Create a Location |
| GET | `/v1/locations/{locationId}` | Get Location By ID |
| PUT | `/v1/locations/{locationId}` | Update a Location |
| DELETE | `/v1/locations/{locationId}` | Delete a Location |

List filters: `name`, `id` (24-char id).

**Create/update body:** GeoJSON **Feature**: either **Polygon** geometry, or **Point** + **properties.subType `Circle`** and **radius** (meters). Required top-level keys include `location` (the feature) and `name`. See OpenAPI `POST /v1/locations/` for examples.

## Schedules

| Method | Path | Summary |
|--------|------|---------|
| GET | `/v1/schedules/` | Get Schedules |
| POST | `/v1/schedules/` | Create a Schedule |
| GET | `/v1/schedules/{scheduleId}` | Get Schedule By ID |
| PUT | `/v1/schedules/{scheduleId}` | Update a Schedule |
| DELETE | `/v1/schedules/{scheduleId}` | Delete a Schedule |

List filters: `id`, `name`.

## Application geo-zones

| Method | Path | Summary |
|--------|------|---------|
| GET | `/v1/application-geozones/` | Get Application Geo-Zones |
| POST | `/v1/application-geozones/` | Create an Application Geo-Zone |
| GET | `/v1/application-geozones/{applicationGeozoneId}` | Get Application Geo-Zone By ID |
| DELETE | `/v1/application-geozones/{applicationGeozoneId}` | Delete an Application Geo-Zone |

List filters: `imei` (15-digit device), `locationId`, `scheduleId`.

## User geo-zones (read-only via API)

| Method | Path | Summary |
|--------|------|---------|
| GET | `/v1/user-geozones/` | Get User Geo-Zones |
| GET | `/v1/user-geozones/{userGeozoneId}` | Get User Geo-Zone By ID |

List filter: `imei`.

## Webhooks (subscription management API)

Distinct from **receiving** webhooks — these endpoints manage webhook configs associated with the authorized user/app.

| Method | Path | Summary |
|--------|------|---------|
| GET | `/v1/webhooks` | Get Webhooks |
| POST | `/v1/webhooks` | Create Webhook |
| PUT | `/v1/webhooks/{webhookId}` | Update Webhook |
| DELETE | `/v1/webhooks/{webhookId}` | Delete Webhook |

Inbound webhook protocol (headers, retries): [webhooks.md](./webhooks.md).

## Errors

Typical HTTP status usage (from official docs):

| Code | Meaning |
|------|---------|
| 200 | GET success |
| 201 | POST created |
| 204 | DELETE success |
| 400 | Bad request; JSON may include `{ "errors": "..." }` |
| 401 | Unauthorized |
| 404 | Not found |
| 50x | Bouncie-side error |

For field-level constraints (path patterns, IMEI length, etc.), rely on [openapi.json](./openapi.json).
