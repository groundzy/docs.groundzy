# Drawer PR checklist

Every PR that adds or materially changes a drawer (under `app/drawers/`) or drawer infrastructure must confirm the following.

**Binary rule:** Each registered drawer is either **DrawerShell** (below) or listed in **[drawer-shell-classification.md](./drawer-shell-classification.md)** — no unclassified custom root layout.

## Layout (DrawerShell contract)

- [ ] Uses **DrawerShell** for the entry root, **or** a full-drawer exception in [drawer-shell-classification.md](./drawer-shell-classification.md) § Exceptions / PR description.
- [ ] Primary column scroll: **DrawerScrollArea** by default, **or** a pattern listed in [drawer-shell-classification.md](./drawer-shell-classification.md) § **Primary column scroll — approved variants** (e.g. Radix `ScrollArea` for chat threads, `ListContainer` for list drawers).
- [ ] **No second outer scroll** for the main column beyond the default or an approved variant above.
- [ ] **No** ad hoc `overflow-y-auto` (or equivalent) on arbitrary wrapper divs for the primary column—scroll belongs in **DrawerScrollArea** or an approved variant.
- [ ] **Do not** add new primary-column scroll mechanisms except by updating § Primary column scroll in `drawer-shell-classification.md` in the same PR.
- [ ] **Footer** (primary actions) is in **DrawerFooter** or outside the scroll region — not fixed/sticky inside scroll content.

## Identity and registry

- [ ] **Registry** ([lib/drawers.ts](lib/drawers.ts)) updated if IDs, labels, order, tiers, or flags change — metadata **matches actual behavior** (comments included).
- [ ] **Multi-ID drawers** (same component, multiple `registerDrawer` ids): **explicit mode** via enum + hook — no scattered `if (drawerId === …)` without a single source of truth.

## Behavior (when applicable)

- [ ] **Drawer `role`** set on new drawers (Phase 4+); see [lib/drawer-registry.ts](lib/drawer-registry.ts).
- [ ] **Workflow / dirty**: forms that can lose data use `useWorkflowFormDirty` + discard dialog when listed in `WORKFLOW_DRAWERS` ([stores/navigation-store.ts](../stores/navigation-store.ts)).

## Copy

- [ ] New user-visible strings use **i18n** (`useI18n` / `t()` and keys in [lib/i18n/messages.ts](../lib/i18n/messages.ts)) where the project already uses i18n for that surface.

## Related docs

- [drawer-navigation.md](./drawer-navigation.md) — history and tool-drawer rules.
- [drawer-audit.md](./drawer-audit.md) — inventory and refactor context.
