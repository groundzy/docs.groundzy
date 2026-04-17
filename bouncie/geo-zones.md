# Bouncie geo-zones

Source: [Bouncie API — Geo-Zones](https://docs.bouncie.dev/#/) and location/application-geozone endpoints in [openapi.json](./openapi.json).

## Two kinds

### User geo-zones

- Created and edited by the **end user** in the Bouncie app or web dashboard (home, work, school, etc.).
- Used for **consumer notifications** when vehicles enter/exit.
- Your integration **cannot modify** them via API.
- You can **read** them: `GET /v1/user-geozones/` and `GET /v1/user-geozones/{userGeozoneId}` (optional `imei` filter on list).

### Application geo-zones

- Created **only through the API** for integrations.
- **Do not appear** in the standard Bouncie consumer experience and **do not** notify end users in the Bouncie client.
- Useful for fleet/workforce rules, server-side logic, or correlating with your own map (e.g. Groundzy job sites).

## Creating an application geo-zone

Recommended sequence from the official docs:

1. **Create a location** — `POST /v1/locations/`  
   Supply a GeoJSON **Feature**:
   - **Polygon** — `geometry.type` = `Polygon`, coordinates per [RFC 7946](https://datatracker.ietf.org/doc/html/rfc7946).
   - **Circle** — `geometry.type` = `Point` with `[lon, lat]`, plus `properties.subType: "Circle"` and `properties.radius` in **meters**.
   - Include a **`name`** for the location.

2. **Optionally create a schedule** — `POST /v1/schedules/`  
   Defines time windows when the geo-zone is active (shape is in OpenAPI).

3. **Create the application geo-zone** — `POST /v1/application-geozones/`  
   Reference the `locationId` and, if used, `scheduleId`, plus device targeting fields as required by the schema (see OpenAPI).

List/query helpers: `GET /v1/application-geozones/` with filters `imei`, `locationId`, `scheduleId`; single resource `GET`/`DELETE` by id.

## Webhook events

Application and user geo-zone transitions can generate webhook events (**Application Geo-Zone**, **User Geo-Zone**) — see [webhooks.md](./webhooks.md).

## Coordinate order

OpenAPI stresses **longitude, then latitude** in coordinate arrays (`[lon, lat]`).
