# Search

**System architecture** (client filtering, scale limits, duplication): [`../07-systems/search-system.md`](../07-systems/search-system.md). This file is the **product/feature** summary; do not duplicate technical audit tables there.

## What it does

**Global search** drawer: query across trees, zones, properties, clients, and species catalog; results grouped by type; navigation opens **view-*** drawers. Debounced input.

## Who uses it

All authenticated users; **clients/properties** results gated by **Pro/Teams** (`useProOrTeamsAccess` per `docs/features/search.md`).

## Data involved

- **Client-side** filtering over loaded lists (`useTrees`, `useZones`, etc.)—not a dedicated Algolia index in the surveyed doc.
- **Species:** `useSearchSpecies(debouncedQuery)`.

## UI patterns

**Exception drawer:** `search` **without** `DrawerShell` (`docs/drawer-shell-classification.md`)—standalone list + `ListContainer`.

## Dependencies

- Entity hooks for all searchable collections
- Tree usage for “add tree” CTA
- Tier hooks

## Inconsistencies & overlaps

- **Search** duplicates **map** discovery (pan vs text)—two mental models for “find a tree.”
- **Species** search overlaps **trees** species autocomplete and **explore**—same catalog, multiple UIs.
