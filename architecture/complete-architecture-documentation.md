# Complete Architecture Documentation

## Architecture Overview

Groundzy is a **map-centric**, **drawer-based** SPA built on Next.js 16 with Firebase as the backend. The UI is organized around a central Mapbox map; users open drawers (panels) for different features. Navigation is URL-driven via `?drawer=<id>&<params>`.

## Routing

### App Router

- **Route groups**: `(auth)` for login, payment success, auth callback
- **Main app**: Single-page layout with map + work area; drawers are not separate routes
- **Share**: `app/share/[token]/` for public share links (token resolved server-side)

### Drawer-Based Navigation

- **URL format**: `?drawer=<drawerId>&treeId=xxx&clientId=yyy`
- **Utilities**: `lib/drawer-utils.ts`
  - `parseDrawerParams(searchParams, isDrawerRegistered)` → `{ drawerId, params }`
  - `buildDrawerUrl(drawerId, params)` → `?drawer=...&...`
  - `drawerIdToUrlParam`, `urlParamToDrawerId`
- **Registry**: `lib/drawer-registry.ts` – lazy-loaded drawers with metadata
- **Registration**: `lib/drawers.ts` – all 40+ drawers

## State Management

### Zustand (Client UI State)

| Store | Purpose |
|-------|---------|
| `navigation-store` | Sidebar open/closed, work area state, drawer history. **Persisted** via `zustand/persist` |
| `map-store` | Map center, zoom, bounds |
| `selection-store` | Selected tree, zone, property |
| `draw-store` | Draw mode, polygon state |
| `measure-store` | Measure tool state |
| `tree-add-store` | Add-tree form state |
| `auth-store` | Auth UI state |
| `fast-mapping-store` | Fast mapping mode |
| `identifying-wand-store` | AI identify wizard |
| `view-tree-header-store` | View-tree header |
| `breakout-placement-store` | Breakout placement |

### TanStack React Query (Server Data)

- Trees, clients, properties, zones, requests, quotes, jobs, invoices
- Hooks in `hooks/` (e.g. `useTrees`, `useClients`)
- Caching, invalidation, optimistic updates

## Data Layer

### Firebase

- **Firestore**: Main database (trees, zones, clients, properties, jobs, invoices, etc.)
- **Firebase Auth**: Authentication
- **Firebase Storage**: Tree images, user gallery, AI chat attachments
- **Firebase Admin SDK**: Server-side (custom tokens, Stripe webhooks, share resolution)

### Access Patterns

- **Individual users**: `databaseCode` (from user doc) scopes trees, clients, etc.
- **Teams**: `organizationId` scopes team data; `members` map for membership
- **Shared trees**: `shared_data/{treeId}` with `sharedWith` array
- **Global admins**: `admins/{uid}` allowlist for admin panel
- **Future**: [Visibility & Permission Model v2](visibility-permission-model-v2.md) — design spec for first-class visibility levels and roles

## Layout and Components

### Layout Hierarchy

1. **app/layout.tsx** – Root layout (providers)
2. **components/layout/app-layout.tsx** – Main layout: sidebar + map + work area
3. **components/work-area/work-area-desktop.tsx** – Desktop work area (drawer panel)
4. **components/work-area/work-area-mobile.tsx** – Mobile work area (bottom sheet)

### Map Components

- **mapbox-map.tsx**: Main map container
- **map-markers.tsx**, **tree-marker.tsx**, **restricted-tree-marker.tsx**: Tree markers
- **zone-layer.tsx**, **property-layer.tsx**: GeoJSON layers
- **draw-control.tsx**, **measure-control.tsx**: Mapbox Draw, measurement
- **map-toolbar.tsx**, **map-top-bar.tsx**: Map controls
- **user-location-marker.tsx**, **center-pointer.tsx**: UX markers

### Drawer Layout Primitives

- **drawer-header.tsx**, **drawer-body.tsx**, **drawer-footer.tsx**
- **list-container.tsx**, **filter-bar.tsx**: List views

## API Layer

- **Next.js API routes**: `app/api/**/route.ts`
- **Server actions**: `app/actions/auth.ts`, `payment.ts`, `team.ts`
- See [reference/api-routes.md](../reference/api-routes.md) for full list

## Subscription Tiers

- **Tiers**: Home, Plus, Pro, Small Team, Mid Team, Large Team, Enterprise
- **Logic**: `lib/utils/tier-utils.ts` – `getUserSubscriptionTier`, `getEffectiveSubscriptionTier`, `hasActivePaidSubscription`
- **Drawer visibility**: `visibleForTiers` in `lib/drawers.ts`
- **Stripe**: Checkout, portal, webhooks; subscription status in user doc

## Internationalization (i18n)

- **Locales**: en, es, fr
- **Config**: `lib/i18n/`, `components/i18n/`
- **Usage**: `useI18n()`, `t(key)` for UI strings
- **Messages**: `lib/i18n/messages.ts`, `lib/i18n/navigation.ts`
- **User preference**: Locale stored in user doc / Firestore

## Security

- **Firestore rules**: `firebase/firestore.rules` – per-collection read/write rules
- **Storage rules**: `firebase/storage.rules`
- **Auth**: Firebase Auth; custom tokens for server-side flows
- **Stripe**: Webhook signature verification; no client-side secret keys
