# Component Standards (Groundzy v3)

Defines **one system** per UI category, **mapping** from shadcn/Radix (implementation) to **Groundzy UI** (product API). Feature code uses **Groundzy names only**.

---

## 1. Naming convention

- **Prefix:** `Gz` (e.g. `GzButton`, `GzDrawerShell`).
- **Package:** e.g. `@/components/groundzy-ui/` or `@groundzy/ui` — repo to finalize.
- **Primitives folder:** shadcn may live under `groundzy-ui/primitives/` or remain in `components/ui` **only** imported by Groundzy wrappers.

---

## 2. Drawers

| Responsibility | Groundzy component | Notes |
|----------------|---------------------|--------|
| Root layout | `GzDrawerShell` | Single shell; **no** duplicate `DrawerShell` variants in features. |
| Primary scroll | `GzDrawerBody` / `GzDrawerScrollArea` | Replaces ad hoc `overflow-y-auto` on wrappers. |
| Sections | `GzDrawerSection` | Title, description, optional actions — consistent spacing. |
| Footer actions | `GzDrawerFooter` | Primary / secondary placement rules. |
| List + filter lists | `GzDrawerList` | Wraps list archetype (filters + scroll list) — replaces one-off ListContainer wiring per feature. |

**Legacy:** `components/drawer-layout/*` informs behavior; v3 **re-exports or replaces** under Groundzy UI so features never import drawer-layout directly.

**Rule:** **No custom drawer implementations** in feature code. Wizards use `GzWizardFrame` (or equivalent) inside the Groundzy package, not a bespoke `div` stack in `app/drawers`.

---

## 3. Modals and dialogs

| Responsibility | Groundzy component | Notes |
|----------------|---------------------|--------|
| Standard modal | `GzModal` | Header, body, footer slots; size tokens (`sm` / `md` / `lg` / `full`). |
| Destructive confirm | `GzConfirmDialog` | Unifies `ConfirmDestructiveDialog`-style flows. |
| Lightweight popover | `GzPopover` | For pickers and contextual actions — one wrapper. |

**Rule:** Features **do not** import `Dialog`, `AlertDialog` from `@/components/ui`. **No** raw `@radix-ui/react-dialog` in features.

---

## 4. Forms

| Responsibility | Groundzy component | Notes |
|----------------|---------------------|--------|
| Field row | `GzFormField` | Label, control, hint, error — consistent layout. |
| Section | `GzFormSection` | Grouped fields with optional title/description. |
| Actions | `GzFormActions` | Sticky or footer-aligned submit/cancel. |
| Validation display | Built into `GzFormField` | Same error styling everywhere. |

**Data:** React Hook Form + Zod stay at the **feature container** or a thin `useXForm` hook; **presentation** is only Groundzy. **No** per-feature `Label`+`Input` stacks from raw ui.

---

## 5. Tables and structured lists

| Responsibility | Groundzy component | Notes |
|----------------|---------------------|--------|
| Card list (CRM pattern) | `GzEntityCard` / `GzDataRow` | Single row/card pattern for requests, jobs, clients, etc. |
| Table (dense) | `GzTable` | Wrapper around table primitives with consistent header/body/empty. |

**Rule:** Workflow list drawers (`requests`, `quotes`, `jobs`, `invoices`) **share** the same list row component family — not copy-paste card markup.

---

## 6. Entity views (tree, client, job, …)

| Responsibility | Groundzy component | Notes |
|----------------|---------------------|--------|
| Detail shell | `GzEntityShell` | Header (title, status, actions), tabs, body region. |
| Tab strip | `GzEntityTabs` | Same tab behavior for view-tree, view-job, view-client, etc. |
| Key-value blocks | `GzDefinitionList` or `GzPropertyGrid` | One pattern for metadata sections. |

**Rule:** **No** entity-specific `Card` composition from `@/components/ui` in feature files — use `GzEntity*` building blocks.

---

## 7. Navigation

| Responsibility | Groundzy component | Notes |
|----------------|---------------------|--------|
| Sidebar item | `GzSidebarNavItem` | Icons, active state, workflow step colors via tokens. |
| Mobile bottom nav | `GzBottomNav` | Consistent with design tokens. |
| Breadcrumbs / back | `GzPageHeader` | Unified back + title pattern for nested flows. |

---

## 8. Rules (CRITICAL)

| # | Rule |
|---|------|
| R1 | **No direct shadcn** — Features may not import from `@/components/ui/*` (or equivalent primitive path). |
| R2 | **No direct Radix in features** — Only Groundzy UI imports Radix/shadcn. |
| R3 | **No custom drawer roots** — One shell family; wizard variants are **Groundzy** components. |
| R4 | **No duplicated UI logic** — Two features needing the same pattern **extract** a Groundzy component or shared hook approved by design system. |
| R5 | **All UI goes through Groundzy** — Including empty states, skeletons, and errors (see [`interaction-patterns.md`](./interaction-patterns.md)). |
| R6 | **Exceptions** — New primitives require **design-system PR** + update to [`design-system.md`](./design-system.md); no silent one-offs. |

---

## 9. Migration stance (from legacy)

Existing `components/ui/*` and `components/drawer-layout/*` are **wrapped** or **aliased** into Groundzy UI, then features migrate import-by-import. Greenfield v3 feature folders start **only** with Groundzy imports.

---

## Related

- [`layout-system.md`](./layout-system.md`)
- [`interaction-patterns.md`](./interaction-patterns.md`)
