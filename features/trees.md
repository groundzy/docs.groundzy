# Trees Feature

Trees (Map Items) are the core entity: individual trees, shrubs, stumps, or zone inventories mapped on the map.

## Overview

- **Drawer ID**: `trees` (list), `tree-add`, `view-tree`, `edit-tree`, `restricted-tree`, `entry-form`, `multiple-add`
- **Collections**: Firestore `trees`, `species_catalog`, `quick_picks`, `quick_pick_sets`, `quick_pick_regions`
- **Tier**: Home+ for list; tree limits apply (Home: 5, Plus/Pro: unlimited)

## Tree Data Model

See `types/tree.ts`:

- **Core**: `lat`, `lng`, `type` (Tree/Shrub/Stump), `commonName`, `species`, `health`, `riskLevel`, `measurements`, `zoneId`, `propertyId`, `clientId`
- **Health**: `overallStatus` (excellent/good/fair/poor/critical/dead), component scores
- **Measurements**: `height`, `dbh`, `crownSpread`, `age`, etc.
- **History**: `history` array – `ServiceRecord`, `InspectionRecord`, `MeasurementRecord`, `NoteRecord`
- **Media**: Photos stored in Firebase Storage `organizations/{orgId}/trees/{treeId}/photos/`
- **Scoping**: `databaseCode` (individual), `organizationId` (team)

## Map Items List (`trees` drawer)

- **Location**: `app/drawers/trees.tsx`, `components/trees/map-items-list.tsx`
- **Filters**: Search, health status, plant type, "Only visible on map" (bounds filter)
- **Bulk actions**: Select multiple → Archive, Share
- **Map sync**: `filterTreesByBounds`, `filterZonesByBounds`, `filterPropertiesByBounds` when "Only visible" is on
- **Add record type**: `AddRecordTypePopover` – add tree, zone, or tree inventory
- **Shared trees**: `ArchiveSharedTreeDialog` for trees shared with others

## Add Tree (`tree-add` drawer)

- **Location**: `app/drawers/tree-add.tsx`, `components/trees/add-tree-form.tsx`
- **Entry modes**: Single tree, Zone, Tree inventory
- **Coordinates**: From URL `lat`/`lng`, or map center; syncs with map movement (debounced)
- **AI Identify flow**: `identifying-wand-store` – pending species/photos applied when returning from AI Identify
- **Form sections** (accordion):
  - Type (Tree/Shrub/Stump), species (SpeciesAutocomplete, QuickPicksChips)
  - Location (lat/lng, property, zone, client)
  - Measurements (height, dbh, crown, age)
  - Health assessment
  - Media (TreeMediaPicker)
  - History entries
- **Quick picks**: `QuickPicksChips` – region-based species chips from `quick_pick_regions`, `quick_pick_sets`, `quick_picks`
- **Validation**: Zod schema, React Hook Form

## View Tree (`view-tree` drawer)

- **Location**: `app/drawers/view-tree/`
- **Params**: `treeId` (required)
- **Tabs**: Overview, History, Share
- **Overview**: Species, health, risk, measurements, property/zone, media, map preview
- **History**: Add entry (measurement, inspection, service, note) via `AddRecordTypePopover`, `AddHistoryEntryForm`
- **Share**: Create share link, grant access to user by username, tree access requests
- **Actions**: Edit, Archive, Delete (soft delete)
- **Restricted trees**: Shown as `restricted-tree` drawer when user has shared access but not full ownership

## Edit Tree (`edit-tree` drawer)

- Same form as Add Tree, pre-filled from existing tree
- **Params**: `treeId`

## Entry Form (`entry-form` drawer)

- **Params**: `treeId`, `entryType`, `entryId?`, `returnTo?`
- Add history entry (measurement, inspection, service, note) for existing tree

## Multiple Add (`multiple-add` drawer)

- Bulk add trees (e.g. from zone or inventory)
- **Location**: `app/drawers/multiple-add.tsx`

## Species & Quick Picks

- **Species catalog**: `species_catalog` collection, `useSpeciesCatalog()`, `SpeciesAutocomplete`
- **Quick picks**: `getMeasurementQuickPicks()`, `getStumpQuickPicks()` for measurement presets
- **API**: `GET /api/quick-picks` – requires Firestore composite indexes

## Tree Limits

- **Home**: 5 trees
- **Plus/Pro/Teams**: Unlimited
- **Enforcement**: `useTreeUsage()`, `LIMITS_COPY.treeLimitReached()` in profile-copy

## Related Hooks

| Hook | Purpose |
|------|---------|
| `useTrees` | List trees by org |
| `useTree` | Single tree by ID |
| `useDeleteTree` | Archive tree |
| `useArchiveTreeForOwnerOnly` | Archive (owner only) |
| `useUnarchiveTreeForOwner` | Unarchive |
| `useTreeUsage` | Usage/limits |
| `useTreeSharedWith` | Shared-with list |
| `useSharedTreeIds` | IDs of trees shared with current user |
| `useSpeciesCatalog` | Species catalog |
| `useSearchSpecies` | Species search |
| `useQuickPicks` | Quick picks by region |

## Archive and public map projection

- **Full archive** (soft delete for everyone): `useDeleteTree` calls `POST /api/trees/archive` (Firebase Admin SDK) so `tree_public`, `tree_public_summaries`, and `community_posts` stay consistent with Firestore security rules (collaborators previously could not update summaries client-side).
- **Owner-only archive** (shared users keep access): `useArchiveTreeForOwnerOnly` calls `POST /api/trees/archive-for-owner`.
- **Safety net**: Cloud Function `onTreePublicLifecycleWritten` repeats public cleanup when `trees/{treeId}` transitions to deleted or owner-archived (idempotent alongside the APIs).
- **Backfill**: Global admins may run `POST /api/trees/cleanup-orphaned-public` for orphaned public projections.
