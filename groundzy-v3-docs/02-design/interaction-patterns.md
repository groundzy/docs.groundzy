# Interaction Patterns (Groundzy v3)

Standard behavior for **loading**, **empty**, **error**, **transitions**, and **navigation** so the product feels unified. All patterns are implemented as **Groundzy UI** components or hooks — features supply **data and copy keys**, not one-off spinners.

---

## 1. Loading states

| Pattern | Groundzy approach | Rules |
|---------|-------------------|--------|
| **Initial load** | `GzSkeleton` blocks matching final layout (list rows, form sections) | No generic centered spinner for full drawer content unless explicitly specified for short async. |
| **Inline refresh** | Subtle `GzLoadingOverlay` or row-level spinners | Same component for CRM lists and tree lists. |
| **Button submit** | `GzButton` `loading` prop | Disables duplicate submit; consistent spinner placement. |

**Anti-pattern:** Per-feature `Loader2` from lucide scattered with different sizes — use **Gz** primitives only.

---

## 2. Empty states

| Pattern | Groundzy approach | Rules |
|---------|-------------------|--------|
| **No data** | `GzEmptyState` — illustration slot optional, title, description, primary CTA | Same vertical stack and spacing for trees, clients, jobs, search. |
| **Filtered empty** | `GzEmptyState` variant `filtered` | Clear filters action + copy pattern. |

**Rule:** Empty states are **not** bespoke `div` + `p` per drawer.

---

## 3. Error states

| Pattern | Groundzy approach | Rules |
|---------|-------------------|--------|
| **Form errors** | Inline under `GzFormField` + toast for submit failure | Same toast styling (sonner or Gz-wrapped). |
| **Section / load failure** | `GzErrorPanel` — title, message, retry | Used for Firestore/query failures in drawer body. |
| **Fatal route** | Full `GzErrorBoundary` fallback | Consistent recovery actions (reload, go home). |

**Rule:** No raw `text-destructive` paragraphs without the **Gz** error pattern.

---

## 4. Transitions and motion

- **Drawer open/close:** Handled by shell/layout — features do not add `animate-*` to roots.
- **List updates:** Prefer CSS transitions on height/opacity via Groundzy list — avoid per-row random animation.
- **Respect `prefers-reduced-motion`:** Groundzy UI implements reduction; features do not add mandatory motion.

---

## 5. Navigation behavior

| Pattern | Behavior |
|---------|----------|
| **Back / forward** | Consistent with app shell; Groundzy header/back components. |
| **Unsaved changes** | Single discard dialog pattern (legacy: `WorkflowFormDiscardDialog` + `WORKFLOW_DRAWERS`) — v3: **GzNavigationGuard** hook + one dialog component. |
| **Deep links** | Same URL-driven drawer model conceptually; loading state while resolving params uses **Gz** skeletons. |

---

## 6. Focus and keyboard

- **Focus trap** in modals: via Groundzy modal only.
- **Tab order** in forms: follows field order in `GzFormField` — no custom `tabIndex` in features.

---

## Related

- [`component-standards.md`](./component-standards.md)
- [`layout-system.md`](./layout-system.md)
- Legacy reference: `docs/audits/drawer-navigation.md`, `stores/navigation-store.ts` (dirty workflow behavior to be reimplemented behind Gz patterns)
