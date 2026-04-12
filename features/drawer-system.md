# Drawer System

The drawer system provides a centralized, lazy-loaded navigation layer for all major screens. Drawers are opened via URL parameters and rendered in the work area (desktop: side panel; mobile: bottom sheet).

## Overview

- **Registry**: `lib/drawer-registry.ts` – stores metadata and lazy-loaded components
- **Registration**: `lib/drawers.ts` – registers all drawers
- **URL format**: `?drawer=<drawerId>&<param1>=<value1>&...`
- **Utilities**: `lib/drawer-utils.ts` – `parseDrawerParams`, `buildDrawerUrl`

## Visual consistency

- **Layout primitives**: `components/drawer-layout/` — `DrawerShell`, `DrawerScrollArea`, `DrawerSection`, `DrawerFooter`, etc. Binary shell vs exception rules: [`docs/drawer-shell-classification.md`](../drawer-shell-classification.md).
- **Card tiers**: Shared Tailwind class strings in [`lib/card-styles.ts`](../../lib/card-styles.ts) — e.g. `contentCardClass` (default glass card via shadcn `Card`), `DrawerSection` (titled sections; composes the same card surface), `listItemCardClass` / `listItemCardTintedClass`, `brandedDeepTealCardClass` (tea-green border + deep-teal fill), `deepTealSurfaceCardClass` (semantic border + deep-teal fill for workflow detail blocks). Prefer these over ad hoc `bg-[hsl(...)]` + border classes in drawer code.
- **Tokens**: Semantic HSL in `app/globals.css` (`:root`, `@theme`). Drawer-oriented aliases `--drawer-canvas` and `--drawer-card` default to the same surfaces as `--deep-teal` / `--card`; Tailwind utilities `bg-drawer-canvas` and `bg-drawer-card` are available when a surface should track those tokens. The **weather** drawer applies `.weather-drawer-gradient` with scoped glass variables for text and cards on the hero gradient (see comments in `globals.css`).
- **Inventory**: Per-drawer shell, scroll, and card classification: [`docs/drawer-visual-inventory.md`](../drawer-visual-inventory.md).

## Drawer Metadata

Each drawer has metadata (`DrawerMetadata`):

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique drawer ID |
| `label` | string | Display label |
| `icon` | ComponentType | Lucide icon component |
| `order` | number | Sort order (lower = earlier) |
| `category` | string? | Group for More drawer (e.g. "data", "workflow", "features") |
| `requiredProps` | (keyof T)[]? | Required URL params (e.g. `["treeId"]`) |
| `defaultProps` | Partial<T>? | Default props |
| `hideFromNav` | boolean? | Not in sidebar/bottom nav (e.g. opened from map) |
| `visibleForTiers` | SubscriptionTier[]? | Tiers that can see this drawer |
| `visibleInSidebar` | boolean? | Show in sidebar (default: true) |
| `visibleInBottomNav` | boolean? | Show in bottom nav (default: true) |
| `visibleInMoreOnly` | boolean? | Only in More drawer, not main sidebar |
| `bottomNavPriority` | number? | Priority for bottom nav (lower = more important) |
| `defaultMobileState` | MobileWorkAreaState? | "full" \| "half" for mobile sheet |
| `extendOverBottomNav` | boolean? | Sheet extends over bottom nav |
| `showHeaderNav` | boolean? | Show back/forward in header (default: true) |
| `archetype` | string? | "informational" \| "form" \| "list" \| "detail" |

## Registry API

```typescript
// Register a drawer
registerDrawer<T>(id: string, metadata: Omit<DrawerMetadata<T>, "id">, importFn: () => Promise<{ default: Component }>)

// Get metadata
getDrawerMetadata(id: string): DrawerMetadata | undefined
getAllDrawerMetadata(): DrawerMetadata[]
getSidebarDrawerMetadata(userTier?: SubscriptionTier | null): DrawerMetadata[]
getBottomNavDrawerMetadata(userTier?: SubscriptionTier | null, maxItems?: number): { items, hasMore, moreItems }
getMoreDrawerMetadata(userTier?: SubscriptionTier | null, maxBottomNavItems?: number): DrawerMetadata[]
getMoreDrawerGroups(...): MoreDrawerGroup[]

// Get component
getDrawerComponent<T>(id: string): LazyExoticComponent<DrawerComponent<T>> | undefined

// Helpers
isDrawerRegistered(id: string): boolean
getRegisteredDrawerIds(): string[]
```

## Adding a New Drawer

1. **Create the drawer component** in `app/drawers/<drawer-id>.tsx`:
   ```tsx
   export function MyDrawer() {
     return <DrawerBody>...</DrawerBody>;
   }
   ```

2. **Register in `lib/drawers.ts`**:
   ```typescript
   registerDrawer("my-drawer", {
     label: "My Drawer",
     icon: SomeIcon,
     order: 5,
     archetype: "informational",
     visibleForTiers: HOME_PLUS_PRO_AND_TEAMS,
     visibleInSidebar: true,
     visibleInBottomNav: false,
     defaultMobileState: "full",
   }, () => import("@/app/drawers/my-drawer").then(m => ({ default: m.MyDrawer })));
   ```

3. **Add i18n label** in `lib/i18n/messages.ts` under `navigation.drawerLabels`:
   ```typescript
   "my-drawer": "My Drawer",
   ```

4. **Navigate** via `buildDrawerUrl("my-drawer", { param: "value" })` or `router.push("?" + buildDrawerUrl("my-drawer"))`

## Drawer with Required Props

For drawers that need params (e.g. `view-tree` needs `treeId`):

```typescript
registerDrawer<{ treeId: string }>("view-tree", {
  label: "View Tree",
  icon: Eye,
  order: 2.6,
  archetype: "detail",
  requiredProps: ["treeId"],
  hideFromNav: true,
  defaultMobileState: "full",
}, () => import("@/app/drawers/view-tree/index").then(m => ({ default: m.ViewTree })));
```

The component receives `params` from the URL via the work area / drawer layout.

## More Drawer

The "More" drawer (`drawer=more`) shows items that don't fit in the main bottom nav, plus items with `visibleInMoreOnly: true`. Section labels come from `MORE_SECTION_LABELS` and `NAV_GROUP_LABELS` in `lib/drawers.ts`.

## Retry on Chunk Load Error

`lib/drawers.ts` uses `withRetry()` for some drawers (e.g. `entry-form`) to retry dynamic imports on `ChunkLoadError` (e.g. after cache clear or HMR).

## Registered Drawers (Quick Reference)

| ID | Label | Tier | Nav |
|----|-------|------|-----|
| dashboard | Dashboard | Home+ | Sidebar, BottomNav |
| weather | Weather | Home+ | Sidebar |
| trees | Map Items | Home+ | Sidebar, BottomNav |
| tree-add | Add Tree | All | Map only |
| multiple-add | Multiple | All | Map only |
| search | Search | All | Map only |
| view-tree | View Tree | All | From map |
| entry-form | Add Entry | All | From view-tree |
| restricted-tree | Restricted Tree | All | From map |
| edit-tree | Edit Tree | All | From map |
| view-zone | View Zone | All | From map |
| clients | Clients | Pro+ | Sidebar, BottomNav |
| view-client (`clientId`, optional `contactId`), add-client, edit-client | Client | Pro+ | From clients |
| properties | Properties | Pro+ | Sidebar, BottomNav |
| view-property, add-property, edit-property | Property | Pro+ | From properties |
| requests, view-request, add-request, edit-request | Requests | Teams | Sidebar |
| quotes, view-quote, add-quote, edit-quote | Quotes | Teams | Sidebar |
| jobs, view-job, add-job, edit-job | Jobs | Teams | Sidebar |
| invoices, view-invoice, add-invoice | Invoices | Teams | Sidebar |
| profile | Profile | Home+ | Sidebar |
| my-photos | My Photos | Home+ | From profile |
| hire-groundzy-pro | Hire a Pro | Home, Plus | Sidebar |
| ai-identifying-wand | Identify | Home+ | More |
| ai-chat | AI Wizard | Home+ | More only |
| tutorial | Tutorials | Home+ | Sidebar |
| help | Help & FAQs | Home+ | Sidebar |
| contact-us | Inbox | Home+ | Sidebar |
| team-settings | Team Settings | Teams | Sidebar |
| draw | Add Zone | All | Map toolbar |
| measure | Measure | All | Map toolbar |
| more | More | Home+ | BottomNav overflow |
