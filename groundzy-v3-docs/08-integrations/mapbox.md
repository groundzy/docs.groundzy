# Mapbox

## What it does

**Maps and geospatial UX:** Renders the map (`mapbox-gl`), drawing tools (`@mapbox/mapbox-gl-draw`), markers/layers, and **geocoding / address autocomplete** (per env doc: token used for maps + geocoding).

## How it’s used

| Area | Connection |
|------|------------|
| **Components** | `components/mapbox-map.tsx`, `components/map/*`, drawers |
| **Libs** | `lib/map-constants.ts`, Turf/h3 for geo math (not Mapbox-branded but adjacent) |
| **Env** | `NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN` |

## Risks & constraints

| Risk | Note |
|------|------|
| **Public token** | `NEXT_PUBLIC_*` is exposed to browsers — **URL restrictions** and **scopes** must be locked down in Mapbox dashboard (standard practice for client maps) |
| **Usage quotas** | Map loads and geocoding calls count against Mapbox billing |
| **Vendor lock-in** | Map style, draw, and layer code are Mapbox-specific; migration = major effort |

## Related

- `Groundzy v3/06-features/map.md`
- `@mapbox/mapbox-gl-draw` in `package.json`
