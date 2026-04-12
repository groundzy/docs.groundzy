# Routing & Navigation (Groundzy → v3)

## Current routing model

### App Router (Next.js)

- **Route groups** for auth flows (`app/(auth)/`, etc.).
- **Main application:** Typically a **single main route** hosting map + work area — drawers are **not** separate Next.js routes in the common pattern (`docs/architecture/complete-architecture-documentation.md`).
- **Public share:** `app/share/[token]/` — server resolves token, renders share view.

### Drawer-based navigation (in-app)

- **URL drives the active drawer:** `?drawer=<drawerId>&treeId=...&clientId=...`
- **Utilities:** `lib/drawer-utils.ts` — `parseDrawerParams`, `buildDrawerUrl`, param mapping.
- **Registry:** `lib/drawer-registry.ts` — metadata, lazy imports, `visibleForTiers`.
- **Sync:** `components/work-area/work-area-content.tsx` reads search params, sets active drawer, tier-gates with `canUserAccessDrawer`.

### History and back/forward

- **`navigation-store`** maintains `drawerHistory`, `goBack` / `goForward` (`docs/drawer-audit.md`).
- **Workflow dirty state:** `WORKFLOW_DRAWERS` + `useWorkflowFormDirty` — discard confirm before leaving (`WorkflowFormDiscardDialog` in layout).

### Mobile vs desktop

- Same URL model; **work area** switches between desktop panel and mobile sheet (`work-area-desktop.tsx`, `work-area-mobile.tsx`).

---

## Data flow (navigation)

1. User clicks nav item → `updateParams` / `navigate` (drawer context) → **URL changes**.
2. Work area **effect** reads URL → loads drawer component → drawer reads **entity ids** from params.
3. Drawer uses **React Query** to load entity data by id.

**Coupling:** Drawer id, URL params, and tier checks must stay consistent with `lib/drawers.ts`.

---

## v3 routing & navigation (target)

### Principles

| Principle | Implementation |
|-----------|----------------|
| **Stable URL contract** | One documented scheme for primary navigation (drawer id + typed params); optional migration to **nested search** params if needed. |
| **Single source of truth** | Prefer **URL** for “where am I”; reduce duplicated history stacks unless required for UX. |
| **Platform navigation module** | Owns `buildPath`, `parsePath`, tier guard, and integration with router — not scattered in each drawer. |
| **Deep links** | Every production drawer state should be **linkable** for support and onboarding. |

### Optional evolution

- **Route-based code splitting** for large areas (e.g. `/settings/*`) if product wants bookmarkable full pages — still composable with map shell when needed.
- **Next.js parallel routes** or **intercepting routes** — only if they simplify modals/share; not required by default.

### Unsaved changes

- **One** `GzNavigationGuard` pattern (conceptual) — central registry of protected surfaces, aligned with domain forms.

### Drawers as system primitive (v3)

For the **main authenticated product** (map + work area), **drawers** are a **first-class navigation primitive**, not an incidental UI wrapper: **feature boundaries**, **deep links** (`?drawer=…` + entity ids), and **tier-gated visibility** are defined in terms of **drawer identity** and URL contract. Shell implementation may change (e.g. Groundzy UI `GzDrawer`), but the **model** stays **drawer-centric** unless product explicitly moves primary flows to full-page routes. Auth, share, and settings may use dedicated routes; the core stewardship loop remains aligned with this pattern.

---

## Related

- [`state-management.md`](./state-management.md)
- [`frontend-architecture.md`](./frontend-architecture.md)
- `lib/drawer-utils.ts`, `lib/drawers.ts` (legacy)
- `docs/drawer-navigation.md` (legacy)
