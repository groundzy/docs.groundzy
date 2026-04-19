# BottomNav audit

Source of truth: code only. Primary file is `app/components/navigation/bottom-nav.tsx`. Supporting code: `app/lib/drawer-registry.ts`, `app/lib/drawers.ts`, `app/lib/drawer-navigation.ts`, `app/hooks/useOrgWorkflowReadPermissions.ts`, `app/lib/participant-work-surface-access.ts`, `app/lib/dashboard-surface-access.ts`, `app/lib/utils/tier-utils.ts`, `app/lib/i18n/navigation.ts`, `app/components/work-area/work-area-mobile.tsx`, `app/components/layout/app-layout.tsx`.

This audit describes only what the code does today; no recommendations.

---

## 1. Mount point and visibility

- Mounted by `app/components/layout/app-layout.tsx` as `<BottomNav />` (one site of use).
- The component itself renders `<nav className="… fixed bottom-0 left-0 right-0 w-full md:hidden …">`. `md:hidden` confines it to **mobile only**; on `md+` viewports it is not visible.
- It uses Next’s `usePathname()` and `useRouter()` from `next/navigation` and is a client component (`"use client"`).
- The drawer registry is loaded for its side-effects via `import "@/lib/drawers"` so all drawers are registered before `getBottomNavDrawerMetadata` runs.

## 2. Layout, slot count, and structure

The nav has a fixed height of `h-20` and adds `paddingBottom: env(safe-area-inset-bottom, 0px)` for iOS safe areas. The inner row is `flex items-center justify-around h-full px-2 -translate-y-1`.

The slot model is encoded in `BottomNav` itself, not in the registry:

- The component requests **3** items from the registry: `getBottomNavDrawerMetadata(userTier, 3)`.
- Those 3 items are split into:
  - `leftNavItems = navItems.slice(0, 2)` — first 2 buttons.
  - `rightNavItems = navItems.slice(2, 3)` — 3rd button (rendered after the middle slot).
- A **middle slot** renders the `participant-work` drawer (label "Work") if its metadata exists in the registry. This slot is rendered for **every user**, regardless of tier or of `participant-work`'s own `visibleInBottomNav: false`. In-drawer access checks are expected to handle non-eligible users (per the comment in code).
- A trailing **Groundzy Wizard** button is rendered as a hard-coded item (not part of the registry-driven list). It opens drawer id `groundzy-wizard`.

The comment in code states the intended layout is `2 left + middle + 1 right + wizard = 4 visible nav buttons`, with the central Add/Search action floating from `MobileMapFloatingAction` on top of the map (note: no component named `MobileMapFloatingAction*` exists in the workspace today; the comment refers to an external floating control, but `BottomNav` itself does not render it). Whenever the participant-work middle slot is shown, the actual rendered count becomes 5 (2 left + Work + 1 right + Wizard).

## 3. Item filtering pipeline

Items returned from `getBottomNavDrawerMetadata(userTier, 3)` are then filtered through three additional client-side gates before being split into left/right:

1. **Org workflow list permission gate**:
   ```
   .filter((item) => !ORG_WORKFLOW_LIST_DRAWER_IDS.has(item.id) || workflowPerms.requests.read)
   ```
   `ORG_WORKFLOW_LIST_DRAWER_IDS` (from `useOrgWorkflowReadPermissions.ts`) is `{"requests", "quotes", "jobs", "invoices"}`. With `bottomNavPriority` values configured in `lib/drawers.ts`, none of these four drawer ids are currently `visibleInBottomNav: true`, so this gate is a no-op for the current registry but is in place for safety / future visibility changes. It uses `requests.read` as a single proxy flag for all four list drawers.

2. **Fieldworker hide-`clients-properties` gate**:
   ```
   .filter((item) => item.id !== "clients-properties"
     ? true
     : !isFieldworkerTeamOrgMember({...})
   )
   ```
   Hides the `clients-properties` drawer for users that `isFieldworkerTeamOrgMember` identifies as fieldworker team-org members. All other ids pass through.

3. **Dashboard surface access gate**:
   ```
   .filter((item) => item.id !== "dashboard"
     ? true
     : canAccessDashboardSurface({...})
   )
   ```
   Hides the `dashboard` drawer if `canAccessDashboardSurface` returns false. All other ids pass through.

The `getBottomNavDrawerMetadata` helper (in `lib/drawer-registry.ts`) itself also filters/sorts:

- Excludes `hideFromNav: true` drawers.
- Excludes drawers with `visibleInBottomNav: false`.
- Applies `visibleForTiers` tier gating (drawers with `visibleForTiers` are hidden when no `userTier` is known).
- Sorts ascending by `bottomNavPriority ?? order ?? 999`.
- `slice(0, maxItems)` gives the visible items; the rest are returned as `moreItems` (ignored by `BottomNav`, used by the More drawer surface).

## 4. Concrete drawer set sourced from `lib/drawers.ts`

Drawers currently flagged `visibleInBottomNav: true` (with their `bottomNavPriority`):

| Priority | Drawer id           | Label    | `visibleForTiers`                                                                  |
|----------|---------------------|----------|------------------------------------------------------------------------------------|
| 1        | `dashboard`         | Dashboard | `Home, Plus, Pro, Small Team, Mid Team, Large Team, Enterprise`                    |
| 2        | `trees`             | Map Items | `Home, Plus, Pro, Small Team, Mid Team, Large Team, Enterprise`                    |
| 3        | `explore`           | Explore   | `Home, Plus, Pro, Small Team, Mid Team, Large Team, Enterprise`                    |
| 4        | `clients-properties`| Clients   | `Plus, Pro, Small Team, Mid Team, Large Team, Enterprise`                          |

With `maxItems = 3`, `getBottomNavDrawerMetadata` returns the first three of those (after tier gate), so:

- **Home tier**: `dashboard`, `trees`, `explore` (clients-properties is tier-gated out).
- **Plus / Pro / Team tiers**: `dashboard`, `trees`, `explore` are returned; `clients-properties` falls into the registry’s `moreItems` (it is the 4th by priority and `maxItems=3`). `BottomNav` therefore does **not** render `clients-properties` in the bottom strip — it is only reachable from sidebar / More.

Drawer-level pieces that interact with bottom-nav layout but are themselves not rendered by `BottomNav`:

- `participant-work` (`visibleInBottomNav: false`, `visibleForTiers: Pro + Team tiers`) — rendered by `BottomNav` as the **middle slot** outside the registry pipeline (always shown if the metadata exists).
- `groundzy-wizard` (`hideFromNav: true`) — rendered by `BottomNav` as the trailing **Wizard button** outside the registry pipeline.

## 5. Click handling and navigation behavior

`handleDrawerClick(drawerId, params?)` is the single click path for the registry-driven slots, the participant-work middle slot, and (via `handleGroundzyWizardClick`) the wizard button:

1. **Registration check**: `if (!isDrawerRegistered(drawerId)) { console.warn(...); return; }`.
2. **Toggle close**: if the clicked drawer is already `activeDrawer` and `mobileWorkAreaState !== "closed"`:
   - Calls `attemptWorkflowNavigate(null)`. If it returns false (workflow drawer with unsaved/dirty state guards), the click is aborted — the drawer stays open.
   - Calls `navigateToPathWithoutDrawer(router, pathname)` which `router.push(pathname ?? "/", { scroll: false })` — clears `?drawer=` from the URL.
   - Sets `setActiveDrawer(null)` and `setMobileWorkAreaState("closed")`.
3. **Open / switch path**:
   - Calls `attemptWorkflowNavigate(drawerId, params || {})`. If false, click is aborted.
   - Calls `navigateToDrawer(router, pathname, drawerId, params || {}, { setActiveDrawer, pushToHistory })`. This:
     - `setActiveDrawer(drawerId)` and `pushToHistory(drawerId, params)` in the navigation store.
     - `router.push(pathname + buildDrawerUrl(drawerId, params), { scroll: false })`.
   - **Mobile sheet height bootstrap**: only when `mobileWorkAreaState === "closed"` before the click, the work-area sheet is opened to `getEffectiveDefaultMobileState(metadata)`:
     - If `metadata.defaultMobileState` is set, that wins.
     - Else `role: "workflow"` → `"full"`, `role: "tool"` → `"half"`, otherwise `"half"` (`drawer-registry.ts` `getEffectiveDefaultMobileState`).

Switching from one open drawer to another therefore preserves the existing `mobileWorkAreaState`; only opening from a closed state uses the per-drawer default.

`navigateToDrawer` and `navigateToPathWithoutDrawer` are imported from `lib/drawer-navigation.ts`. `BottomNav` does not call `router.replace` or `replaceDrawerUrlParams`.

The wizard button uses `handleGroundzyWizardClick = () => handleDrawerClick("groundzy-wizard")`, with no params.

## 6. Z-index relationship to drawers

```ts
const activeDrawerMeta = activeDrawer ? getDrawerMetadata(activeDrawer) : undefined;
const drawerExtendsOverNav = activeDrawerMeta?.extendOverBottomNav === true;
const navZIndex = drawerExtendsOverNav ? "z-40" : "z-[60]";
```

When the active drawer’s metadata sets `extendOverBottomNav: true`, the bottom nav drops to `z-40` so the drawer sheet (anchored to viewport bottom by `work-area-mobile.tsx`, see `BOTTOM_NAV_OFFSET` and `STATE_HEIGHTS_EXTENDED`) can render on top of it. Otherwise the nav floats above the sheet at `z-[60]`. The mobile sheet logic in `components/work-area/work-area-mobile.tsx` reads the same `extendOverBottomNav` flag to decide whether the sheet `bottom` is `0` or `BOTTOM_NAV_OFFSET`.

## 7. Active state, badges, and i18n

- `isActive = activeDrawer === item.id` is computed for each rendered button (including the participant-work middle slot and the wizard, which compares to `"groundzy-wizard"` directly).
- Active styling on `NavItemButton`:
  - Outer button gets `bg-accent`.
  - Inner icon container swaps from `nav-icon-deep-teal` to `bg-[hsl(var(--green-yellow))] text-[hsl(var(--deep-teal))]`.
  - The icon is rendered at `w-7 h-7`, `strokeWidth={2.5}`.
- **Inbox unread dot**: any rendered nav item whose id is `contact-us` shows a small destructive dot via `showUnreadBadge` when `hasInboxUnread` is true. `hasInboxUnread = hasUnreadMessages || hasUnreadNotifications || hasPendingAccessRequests`.
  - In the current registry, `contact-us` has `visibleInBottomNav: false`, so this dot path is never reached unless a future configuration change makes `contact-us` appear in the bottom strip. The wiring is present for both `leftNavItems` and `rightNavItems` maps (`item.id === "contact-us" && hasInboxUnread`).
- **Unread data sources**:
  - `useUnreadMessages()` and `useUnreadNotifications()` hooks return `{ hasUnread }`.
  - `usePendingAccessRequestsForOwner(organizationId)` returns `{ data }`. `organizationId` resolution order: `organization?.id` → `userDoc?.organizationId` → `user?.uid`, each accepted only when `typeof === "string"`. `pendingAccessRequests = data ?? []`; `hasPendingAccessRequests = pendingAccessRequests.length > 0`.
- **Labels (i18n)**: `getLocalizedDrawerLabel(item.id, item.label, t, userTier)` is used for `aria-label` on registry-driven buttons and the participant-work middle slot. The wizard button passes a hard-coded fallback string `"Groundzy Wizard"` but still goes through `getLocalizedDrawerLabel("groundzy-wizard", "Groundzy Wizard", t, userTier)`.
- The dot uses `aria-hidden`; buttons set both `aria-label` and `aria-pressed={isActive}`.

## 8. Tier resolution for visibility

`userTier = getEffectiveSubscriptionTier(userDoc)` is the input to:

- `getBottomNavDrawerMetadata(userTier, 3)` (registry tier gating + ordering),
- `getLocalizedDrawerLabel(item.id, item.label, t, userTier)` (label localization may use tier).

`organizationId` is computed (with the order in §7) and passed only into `usePendingAccessRequestsForOwner`. The two custom filter steps (`isFieldworkerTeamOrgMember`, `canAccessDashboardSurface`) consume `{ uid, userDoc, organization }` directly, not `userTier`.

`useAuth()` is the source for `user`, `userDoc`, and `organization`.

## 9. Keys, accessibility, and DOM concerns

- `key={item.id}` is used for both `leftNavItems` and `rightNavItems` map outputs and for the participant-work middle slot. The wizard button is a single static element.
- Each registry-driven button is a single `<button>` with `aria-label` and `aria-pressed`. The wizard button mirrors that pattern but does not use `NavItemButton`; it duplicates the same Tailwind classes inline.
- `relative inline-flex size-12 items-center justify-center rounded-xl` on the icon container is shared across `NavItemButton` and the wizard button.
- `overflow-visible` is set on both the `<nav>` and the inner row, allowing the `-translate-y-1` icons and the `-top-0.5 -right-0.5` unread dot to render outside the strict `h-20` box.
- The class `gz-shell-block-pointer` is applied to the outer `<nav>`; `BottomNav` does not define behavior for it (defined elsewhere in the global CSS).

## 10. Summary of inputs and effects

**Inputs read by `BottomNav`:**

- Navigation store: `activeDrawer`, `mobileWorkAreaState`, `setActiveDrawer`, `setMobileWorkAreaState`, `pushToHistory`, `attemptWorkflowNavigate`.
- Routing: `useRouter()`, `usePathname()`.
- Auth: `user`, `userDoc`, `organization` from `useAuth()`.
- Permissions / surface access: `useOrgWorkflowReadPermissions`, `isFieldworkerTeamOrgMember`, `canAccessDashboardSurface`.
- Counters: `useUnreadMessages`, `useUnreadNotifications`, `usePendingAccessRequestsForOwner`.
- Tier: `getEffectiveSubscriptionTier(userDoc)`.
- i18n: `useI18n` (`t`), `getLocalizedDrawerLabel`.
- Drawer registry: `getBottomNavDrawerMetadata`, `getDrawerMetadata`, `isDrawerRegistered`, `getEffectiveDefaultMobileState`.

**Effects produced:**

- Mutates the navigation store (`setActiveDrawer`, `pushToHistory`, `setMobileWorkAreaState`).
- Pushes URL changes via Next router (`router.push`, never `router.replace`) using `buildDrawerUrl`.
- Renders 4 buttons in the typical case (2 left + Work + 1 right + Wizard), 3 when the participant-work middle slot is suppressed (only possible if `getDrawerMetadata("participant-work")` is undefined; tier/preset do not suppress it in the bottom strip).
- Adjusts its own `z-index` based on the active drawer’s `extendOverBottomNav` flag.

---

This document reflects the code as currently committed in `app/components/navigation/bottom-nav.tsx` and the supporting modules listed at the top.
