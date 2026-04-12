# DrawerShell classification (binary rule)

**Rule:** Every drawer registered in [`lib/drawers.ts`](../lib/drawers.ts) must be in **exactly one** of two states:

1. **DrawerShell** — Root layout uses [`DrawerShell`](../components/drawer-layout/drawer-shell.tsx) from `@/components/drawer-layout` (often with [`DrawerScrollArea`](../components/drawer-layout/drawer-scroll-area.tsx)), per [`DRAWER_PR_CHECKLIST.md`](./DRAWER_PR_CHECKLIST.md).
2. **Documented exception** — Listed in **§ Exceptions** below with the **approved pattern** and reason.

There is **no third state**: custom root layout without an exception row is not allowed for new work; existing gaps should be classified when touched.

**Detail drawers** may use a dedicated layout wrapper that itself uses `DrawerShell` (e.g. [`ViewTreeDrawerLayout`](../app/drawers/view-tree/ViewTreeDrawerLayout.tsx) for `view-tree`).

---

## Primary column scroll — approved variants

- **Default:** [`DrawerScrollArea`](../components/drawer-layout/drawer-scroll-area.tsx).
- **Allowed exceptions:** listed in the table below.

Do not introduce new scroll implementations for the primary column outside this list (new patterns require updating this section in the same PR).

| Pattern | Where | Rationale |
|--------|--------|-----------|
| **Radix `ScrollArea`** | [`app/drawers/ai-chat/AiChatThread.tsx`](../app/drawers/ai-chat/AiChatThread.tsx) | Message list / chat UX. |
| **Radix `ScrollArea`** | [`app/drawers/contact-us/ContactUsThread.tsx`](../app/drawers/contact-us/ContactUsThread.tsx) | Conversation list inside inbox. |
| **`ListContainer`** | [`app/drawers/trees.tsx`](../app/drawers/trees.tsx), [`app/drawers/clients-properties.tsx`](../app/drawers/clients-properties.tsx) | List archetype with `FilterBar` + scrollable list via [`ListContainer`](../components/drawer-layout/list-container.tsx) (still under `DrawerShell`). |

**Related:** the `search` row in **§ Exceptions** uses `FilterBar` + `ListContainer` without `DrawerShell` (full-drawer exception). **`trees`** and **`clients-properties`** use `DrawerShell` with **`ListContainer`** for the list column only.

---

## Early returns without `DrawerShell`

Some branches render **upgrade-gated** UI (Teams/Pro prompts), **loading** spinners, or **not-found** stubs as a minimal centered `div` **without** wrapping in `DrawerShell`. The **normal** content path still uses `DrawerShell` where applicable. When adding or changing such branches, keep them minimal; do not use this as a way to ship a full alternate layout without a **§ Exceptions** row.

---

## Exceptions (approved patterns)

| Drawer ID(s) | Entry file | Approved pattern | Notes |
|--------------|------------|------------------|--------|
| `search` | [`app/drawers/search.tsx`](../app/drawers/search.tsx) | `FilterBar` + `ListContainer` | Search surface; scroll lives in `ListContainer`; no `DrawerShell`. |
| `tree-add` | [`app/drawers/tree-add.tsx`](../app/drawers/tree-add.tsx) | `div` (`h-full flex flex-col overflow-hidden`) + `AddTreeForm` | Thin host; form owns inner layout. |
| `edit-tree` | [`app/drawers/edit-tree.tsx`](../app/drawers/edit-tree.tsx) | `div` + `AddTreeForm` | Same as tree-add for edit mode. |
| `multiple-add` | [`app/drawers/multiple-add.tsx`](../app/drawers/multiple-add.tsx) | `div` + `DrawerBody` / `DrawerFooter` | Map workflow; footer actions outside classic Shell stack. |
| `ai-identifying-wand` | [`app/drawers/ai-identifying-wand.tsx`](../app/drawers/ai-identifying-wand.tsx) | `div` (`h-full min-h-0 flex flex-col overflow-hidden`) + `SpeciesIdentifierWizard` | Full-bleed wizard; embedded component owns chrome. |
| `groundzy-wizard` | [`app/drawers/groundzy-wizard.tsx`](../app/drawers/groundzy-wizard.tsx) | Custom tab shell + embedded flows (`SpeciesIdentifierWizard`, `AiChat`) | Mobile wizard; not a standard detail/form drawer. |

All **other** `registerDrawer` IDs use **`DrawerShell`** at the entry component root (or a wrapper that renders `DrawerShell`), unless this table is updated.

---

## Line budget (Phase C)

Target **~400 lines** for top orchestration entry files (`contact-us`, `request-form`, `view-zone`, `ai-chat`, `profile`). Prefer extracting sections before adding features when a file is already above the budget. Non-blocking CI warning: `npm run verify:drawers` (includes **nested** drawer entry files listed in `NESTED_ENTRY_FILES` in [`scripts/drawer-pr-check-warn.mjs`](../scripts/drawer-pr-check-warn.mjs) — add a path there when you register a new `import("@/app/drawers/.../index")` in [`lib/drawers.ts`](../lib/drawers.ts)).

Recent Phase C work extracted presentational sections under colocated folders while keeping each entry as a **composition root** with `DrawerShell` unchanged — **no new exception rows** were required (see `docs/drawer-refactor-second-pass-retrospective.md` §9).

---

## Related

- [`DRAWER_PR_CHECKLIST.md`](./DRAWER_PR_CHECKLIST.md) — PR requirements for drawer changes (includes primary-column scroll and § Primary column scroll above).
- [`drawer-audit.md`](./drawer-audit.md) — Registry inventory.
