# Drawer navigation and history

This document is the **spec** for how drawer navigation, URL state, and in-app history interact. **Author and update this before large changes to** [`stores/navigation-store.ts`](../stores/navigation-store.ts) or URL sync in [`components/work-area/work-area-content.tsx`](../components/work-area/work-area-content.tsx).

## Source of truth

- **URL** (`?drawer=…` and query params) is the **source of truth** for which drawer is open and with which params.
- **No `drawer` query param** means the drawer is **closed** in the UI: `work-area-content` clears `activeDrawer` and work-area chrome to match the URL (except the initial `/` load path that replaces with `dashboard`).
- [`navigation-store`](../stores/navigation-store.ts) tracks **activeDrawer**, **mobile work area**, **drawer history stack** (`drawerHistory`, `historyIndex`), and **workflow dirty** state.

## Work area header context trail (breadcrumb-style)

The **desktop shell header** and **mobile work area header** can show an optional **single-line context trail** (e.g. client · property · workflow document `#`) derived from the **same URL query params** as the active drawer. Labels load via the **same React Query hooks** as drawer bodies (`useClient`, `useProperty`, `useTree`, workflow getters, etc.), so there is no duplicate fetch layer.

- **Implementation (app codebase):** `hooks/useWorkAreaHeaderContext.ts` (data), `lib/work-area-header-context.ts` (pure segment assembly), `components/work-area/work-area-header-title.tsx` (presentation).
- **Interaction:** Segments are **presentational only** (not clickable); navigation remains sidebar, map, links inside drawers, and workflow discard rules unchanged.
- **Trees:** For `view-tree` / `edit-tree`, resolved client/property precedes the **species** label when applicable; species stays the emphasized trailing segment when the existing scroll-based header logic shows it; when the in-drawer title is visible, the trail ends with the localized drawer title.
- **i18n:** Separator string `workArea.headerContextSeparator` (default middle dot).

## History stack (`pushToHistory` / `syncHistoryFromUrl`)

- Opening a drawer via in-app navigation typically **pushes** a new entry (see `pushToHistory`) so the store can **mirror** browser history for optimistic URL params and short cross-drawer transitions in `work-area-content`. **Users** navigate back and forward with the **browser** only; there is no duplicate back/forward control in the work-area header.
- **`syncHistoryFromUrl`**: when the URL’s drawer + params do not match any in-app stack entry, the stack is **reset** to a single entry for the current URL (avoids duplicate/growing stacks on param key order or deep links). Param equality uses stable comparison (`areDrawerParamsEqual` in `lib/drawer-utils.ts`), not `JSON.stringify`.
- **Browser back** is coordinated with URL sync in `work-area-content`; workflow forms may intercept when dirty.
- **Decisions (current product intent):**
  - **view-tree from map / marker:** **Push** onto the stack (same as other navigations) so internal state stays aligned when the user uses **browser Back** to return to the prior drawer/context; URL remains source of truth. Prefer **replace** only when the same drawer id is reopened with different params if we need to avoid duplicate stack entries (not implemented as a special case today).
  - **Context switch** (e.g. primary list → workflow list): **Append** to the stack; do not auto-reset when switching categories. Reset/fork is reserved for explicit product flows (e.g. closing work area), not for every cross-list navigation.

## Tool drawers (`draw`, `measure`, …)

- **Map-triggered**, often **half-sheet** on mobile; may use `extendOverBottomNav` or special cases in `work-area-content` (e.g. **draw** without pending polygon on narrow viewports).
- **URL**: drawer id and params remain in the URL while open (single source of truth).
- **Mental model:** Tools are **peer drawers** in the stack (same push/sync rules as other drawers). **Draw** without a completed polygon may keep the sheet **closed** on narrow viewports so the map stays usable; that is **layout/sheet** behavior, not a separate history mode.

## Ephemeral / `hideFromNav` drawers

- Not listed in sidebar/bottom nav but still **addressable by URL** and the **same stack** when navigated programmatically.

## Workflow URL params (tree-centric)

- **`add-request`**: optional `clientId`, `propertyId`, `contactId`, `zoneId`, **`treeId`** (prefills `requestItems` with that tree when creating).
- **`add-quote`**: optional `clientId`, `propertyId`, `contactId`, `requestId`, **`treeId`** (when not converting from a request, prefills a line item with that tree).
- **`add-job`**: optional `clientId`, `propertyId`, `contactId`, `requestId`, `quoteId`, **`treeId`** (when not converting from quote/request, prefills a line item with that tree).

These params are passed through `navigate(drawerId, params)`; the URL remains the source of truth (see `lib/drawer-utils.ts`).

- **view-tree (Teams):** The sticky footer puts **Create workflow** (request / quote / job / invoice) as the primary control; **Add Record** and **Schedule** are omitted so tree-level actions match the CRM workflow. Other tiers keep **Add Record** and **Schedule** in the footer unchanged.

## Workflow dirty navigation

- Drawers listed in `WORKFLOW_DRAWERS` participate in **unsaved changes** flow when `workflowFormDirty` is true (`attemptWorkflowNavigate`, discard dialog). The set includes **`draw`** (finish-zone form after drawing a polygon) when `useWorkflowFormDirty("draw", …)` reports dirty.
- See [navigation-store.ts](../stores/navigation-store.ts) for the canonical list.
- **Sub-flow bypass:** If navigation params include `returnTo` equal to the **current** `activeDrawer` (e.g. `add-request` → `add-property` with `returnTo=add-request`), `attemptWorkflowNavigate` **allows** the navigation without opening the discard dialog. Call sites that open add-client / add-property from workflow forms already pass this. Close, sidebar, and other leaves either omit `returnTo` or use a different target, so they still trigger the confirm when dirty.

## Drawer `role` and mobile default sheet

- Registrations may omit `defaultMobileState`; mobile then uses `getEffectiveDefaultMobileState` in [`lib/drawer-registry.ts`](../lib/drawer-registry.ts): **`workflow` → `full`**, **`tool`** and others default to **`half`** unless explicitly set.

## Registry & metadata (keep in sync with code)

- **Registrations** live in [`lib/drawers.ts`](../lib/drawers.ts): each drawer calls `registerDrawer(id, metadata, importFn)`. Types and runtime helpers (`getDrawerMetadata`, nav filters, `getEffectiveDefaultMobileState`) are in [`lib/drawer-registry.ts`](../lib/drawer-registry.ts).
- **Changing behavior or visibility** (tier gates, sidebar/bottom nav, mobile sheet): update the drawer’s `registerDrawer` entry and verify consumers (`getSidebarDrawerMetadata`, `work-area-content`, etc.). Treat the registration as part of the feature — avoid “works in UI but wrong metadata.”
- **`order`**: Sorts nav surfaces; fractional values group items (see comments in `lib/drawers.ts`). Multiple drawers may share the same `order`; use category + product intent. If two items collide unintentionally, fix or document in the PR.
- **Labels**: `label` on metadata is the fallback English string; translated nav labels live in [`lib/i18n/navigation.ts`](../lib/i18n/navigation.ts) (`navigation.drawerLabels`) where keys exist.
- **Inventory / audit**: [`drawer-audit.md`](./drawer-audit.md) lists registered ids and notes; refresh or cross-check when adding drawers or changing metadata.

## Manual QA (regression checks)

After changing URL sync, history, or nav helpers, verify:

- Browser **Back** / **Forward** with a drawer open: URL and visible drawer stay aligned.
- **Dirty workflow** form: sidebar switch to another drawer prompts discard; **bottom nav** toggle-close on the same item prompts discard.
- **Dirty workflow** form: **Add client** / **Add property** (or other sub-flows with `returnTo` = current drawer) does **not** prompt discard; **Close** still should.
- **Map** → **view-tree** (or similar) → **browser Back** returns to the prior drawer/context.
- Remove **`drawer`** from the URL (edit address bar or navigate to `/` without query): drawer UI closes.
- **Deep link** with full params (`?drawer=…&…`): correct drawer opens and in-app history resets to that entry when it did not match the prior stack.
- **Work area header trail:** Open `view-property`, `view-client`, `view-tree`, and a workflow detail drawer (`view-request`, `view-quote`, `view-job`, `view-invoice`) with typical params; confirm the header shows **client · property** (and optional `#`) without duplicate fetch errors; trail is non-interactive.

## Related

- [DRAWER_PR_CHECKLIST.md](./DRAWER_PR_CHECKLIST.md)
- [drawer-audit.md](./drawer-audit.md)
