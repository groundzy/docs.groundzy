# Map and Zones Feature

The map is the central UI. Users view trees, zones, and properties; draw zones; and measure distance/area.

## Overview

- **Map**: Mapbox GL (`mapbox-gl`), `NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN`
- **Components**: `components/map/`, `components/mapbox-map.tsx`
- **Stores**: `map-store`, `selection-store`, `draw-store`, `measure-store`

## Map Components

| Component | Purpose |
|-----------|---------|
| `mapbox-map.tsx` | Main map container, initializes Mapbox |
| `map-markers.tsx` | Tree markers container |
| `tree-marker.tsx` | Individual tree marker |
| `restricted-tree-marker.tsx` | Restricted/shared tree marker |
| `zone-layer.tsx` | Zone polygons (GeoJSON) |
| `property-layer.tsx` | Property polygons |
| `map-toolbar.tsx` | Tools, filters, style, add tree, export |
| `map-top-bar.tsx` | Top bar (search, location) |
| `draw-control.tsx` | Mapbox Draw for polygon drawing |
| `measure-control.tsx` | Distance/area measurement |
| `draw-helper.tsx` | Draw mode helper UI |
| `user-location-marker.tsx` | User GPS position |
| `center-pointer.tsx` | Center crosshair |
| `zoom-level-indicator.tsx` | Zoom level display |
| `zone-context-menu.tsx` | Right-click zone menu |
| `property-context-menu.tsx` | Right-click property menu |
| `map-marker-context-menu.tsx` | Right-click tree marker menu |

## Map Tools (`map-toolbar`)

| Tool | ID | Description |
|------|-----|-------------|
| Move/Select | `default` | Pan, zoom, click to select tree/zone/property |
| Multi-select | `multiSelect` | Select multiple trees for bulk actions |
| Measure | `measure` | Distance (line) or area (polygon) |

## Map Filters

| Filter | Type | Tier |
|--------|------|------|
| Markers | `trees` | All |
| Labels | `showTreeNumbers` | All |
| Zones | `zones` | All |
| Properties | `properties` | Pro+ |
| Clients | `clients` | Pro+ |
| Jobs | `jobs` | Teams |

## Marker zoom (subscription tier)

Own trees, property pins, and restricted-tree markers use tier-aware thresholds from `getMapMarkerZoomParams()` in `lib/utils/marker-zoom-size.ts`. **Other users’ trees** still honor Firestore `TreePublicSummary.minZoom` (e.g. public 14, private 18); tier does not change that.

| Tier | Min zoom to show markers | Unified “x” icon (inclusive) |
|------|----------------------------|------------------------------|
| Home, Plus | 12 | 15 |
| Pro, Teams (active subscription) | 10 | 13 |

Marker size scales between the tier’s minimum zoom and zoom 18. Map PNG export uses the same unified-icon cutoff and skips drawing own/unrestricted markers when zoom is below the tier minimum (parity with on-map visibility).

## Map Styles

- **Cycle**: Streets → Satellite → Hybrid (from `MAP_STYLE_CYCLE` in `lib/map-constants.ts`)
- **Storage**: User preference (persisted)

## Draw Zone (`draw` drawer)

- **Drawer ID**: `draw`
- **Location**: `app/drawers/draw.tsx`
- **Flow**:
  1. User activates Draw tool on map, draws polygon
  2. When polygon complete, `draw` drawer opens with `pendingZoneCoordinates`
  3. User enters name, description, optional property, optional tree counts (species + count)
  4. Save creates zone in Firestore `zones`
- **Store**: `draw-store` – `pendingZoneCoordinates`, `pendingZoneColor`, `pendingPropertyId`, `zones` (in-memory during draw)
- **DrawControl**: `onZoneDrawn` callback passes coordinates; opens drawer via navigation
- **Zone colors**: Green, Blue, Red, Yellow, Orange, Purple
- **Tree limit**: Check `useTreeUsage()`; upgrade CTA if at limit

## Measure (`measure` drawer)

- **Drawer ID**: `measure`
- **Location**: `app/drawers/measure.tsx`
- **Modes**: Line (distance), Area (polygon)
- **Store**: `measure-store` – `mode`, `points`, `distance`, `area`, `history`
- **Units**: Imperial (ft, sq ft) or Metric (m, sq m) from `useUserPreferences().isImperial`
- **Format**: `formatLengthMeters()`, `formatAreaSqMeters()` in `lib/utils/format-units.ts`
- **History**: Push current measurement to history, clear current or history

## Zones

- **Collection**: Firestore `zones`
- **Fields**: `organizationId`, `coordinates` (polygon points), `name`, `description`, `propertyId`, `clientId`, `color`, `speciesCounts`, `media`, `condition`, `measurements`, `workObjectives`, `importSource`, `needsBoundaryReview`, `isDeleted`, `createdAt`
- **Subcollection**: `zones/{zoneId}/zone_services` – bulk zone services
- **Hooks**: `useZones`, `useZone`, `useCreateZone`
- **Map**: `zone-layer.tsx` renders zones; click opens `view-zone` drawer
- **Grouped-tree model**:
  - `speciesCounts` are aggregate trees in the zone without individual `Tree` documents.
  - `tree.zoneId === zone.id` means an individual tree is intentionally assigned to the zone.
  - Trees inside the polygon but without `zoneId` are boundary candidates that can be assigned.

## View Zone (`view-zone` drawer)

- **Params**: `zoneId`
- **Location**: `app/drawers/view-zone.tsx`
- Shows a zone passport plus tabs for overview, inventory, activity, operations, media, and system metadata.
- **Inventory** separates aggregate counts, assigned individual trees, and inside-boundary candidates.
- **Activity/Ops** include zone services, workflow work items, recommended work, and grouped service scheduling.

## Map Store (`map-store`)

| State | Purpose |
|-------|---------|
| `center` | Map center { lat, lng } |
| `zoom` | Zoom level |
| `bounds` | Visible bounds |
| `hasValidBounds` | Bounds loaded |
| `lastBoundsUpdate` | Timestamp |
| `hoveredTreeId` | Hovered tree for highlight |
| `activeTool` | default, multiSelect, measure |
| `requestCenterOn` | Request map to center on coords |
| `mapStyle` | Current style |

## Selection Store (`selection-store`)

| State | Purpose |
|-------|---------|
| `selectedTreeId` | Single selected tree |
| `selectedTreeIds` | Multi-selected trees |
| `selectedZoneId` | Selected zone |
| `selectedPropertyId` | Selected property |
| `toggleTreeSelection`, `clearSelection`, `addTreesToSelection` | Actions |

## Geocoding

- **Forward**: `lib/utils/mapbox-forward-geocode.ts` – address → coords
- **Reverse**: `lib/utils/mapbox-reverse-geocode.ts` – coords → address
- **Address autocomplete**: `components/settings/address-autocomplete.tsx`

## Export Map

- **Toolbar**: Export button triggers `onExportMap`
- Uses `html2canvas` for map screenshot (or similar)
