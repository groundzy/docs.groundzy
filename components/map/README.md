# Map Components

Mapbox map and related components in `components/map/`.

## Core

| Component | Purpose |
|-----------|---------|
| mapbox-map.tsx | Main map container, Mapbox GL init |
| map-markers.tsx | Tree markers container |
| tree-marker.tsx | Individual tree marker |
| restricted-tree-marker.tsx | Restricted/shared tree marker |
| zone-layer.tsx | Zone polygons (GeoJSON) |
| property-layer.tsx | Property polygons |
| property-marker.tsx | Property marker |

## Controls

| Component | Purpose |
|-----------|---------|
| map-toolbar.tsx | Tools, filters, style, add tree, export |
| map-top-bar.tsx | Search, location |
| draw-control.tsx | Mapbox Draw for polygons |
| measure-control.tsx | Distance/area measurement |
| draw-helper.tsx | Draw mode helper UI |

## Context Menus

| Component | Purpose |
|-----------|---------|
| zone-context-menu.tsx | Right-click zone |
| property-context-menu.tsx | Right-click property |
| map-marker-context-menu.tsx | Right-click tree marker |
| map-empty-context-menu.tsx | Right-click empty map |

## Other

| Component | Purpose |
|-----------|---------|
| user-location-marker.tsx | User GPS position |
| center-pointer.tsx | Center crosshair |
| zoom-level-indicator.tsx | Zoom level |

See [features/map-and-zones.md](../../features/map-and-zones.md) for full documentation.
