# Map

## What it does

Central **Mapbox GL** map: tree markers, zone and property polygons, filters, style cycle, zoom-aware marker visibility, **draw zone** flow, **measure** distance/area, toolbar actions (add tree, export, etc.). Map state (center, zoom, bounds) drives many features.

## Who uses it

All tiers (**Home** through **Teams**). **Filter affordances** differ: e.g. properties/clients/jobs filters are **Pro+** or **Teams** per `docs/features/map-and-zones.md`. Marker **min-zoom** thresholds differ by tier (`map-and-zones.md`, `lib/utils/marker-zoom-size.ts`).

## Data involved

- **Read:** Trees, zones, properties (and job markers for Teams) via React Query; public summaries for other users’ trees where applicable.
- **Write:** Zones via `draw` drawer → `zones` collection; not a single “map document”—map is a **view** over entities.
- **Stores:** `map-store`, `selection-store`, `draw-store`, `measure-store` (Zustand).

## UI patterns

- **Shell:** Map always visible in main layout; drawers slide over **work area**, not replacing map (`components/layout/app-layout.tsx`).
- **Exceptions:** `draw` drawer uses documented shell exception (`docs/drawer-shell-classification.md`).
- **Context menus:** Tree, zone, property right-click menus (`components/map/`).

## Dependencies

- Mapbox token `NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN`
- Turf / h3 for geo helpers
- Tier utils, tree usage limits for add-tree

## Inconsistencies & overlaps

- **Filter tier matrix** vs **README tier table** (tree limits)—verify in app for current limits.
- **Jobs on map** (Teams) overlaps **workflow** feature—same `jobs` data, two surfaces (map filter vs jobs list).
- **Draw zone** ties to **trees** (species counts) and **zones** entity—boundary between **map tool** and **inventory** is UX-heavy.
