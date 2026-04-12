# Search Feature

Global search across trees, zones, properties, clients, and species catalog.

## Overview

- **Drawer ID**: `search`
- **Location**: `app/drawers/search.tsx`
- **Opened from**: Map toolbar (search icon)
- **hideFromNav**: true (not in sidebar/bottom nav)

## Search Scope

| Entity | Fields searched |
|--------|-----------------|
| Trees | nickname, species commonName, species scientificName, location address |
| Zones | name, description |
| Properties | address |
| Clients | display name, companyName, firstName, lastName, emails, phones |
| Species | Catalog search via `useSearchSpecies(debouncedQuery)` |

## Behavior

- **Debounce**: 300ms (`DEBOUNCE_MS`)
- **Query**: Trimmed, lowercased
- **Results**: Grouped by entity type; click navigates to view drawer (view-tree, view-zone, view-property, view-client)
- **Add Tree**: Quick action; checks tree limit, upgrade CTA if at limit
- **Pro/Teams**: Clients and properties only shown if `useProOrTeamsAccess()` has access

## Hooks

- `useTrees`, `useZones`, `useProperties`, `useClients` – entity lists
- `useSearchSpecies` – species catalog search
- `useTreeUsage` – tree limit for add-tree CTA
- `useSpeciesDisplayName` – localized species names in results
