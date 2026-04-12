# Workflow discard confirm: double-tap / first tap “only dims” (handoff & resolution)

## Status: **Resolved**

The issue is **fixed** in production. First tap on **Keep editing**, **Discard**, **X**, or the backdrop now completes the intended action.

---

## Original symptom

- On **dirty workflow forms**, the “Discard unsaved changes?” confirm sometimes required **two taps**: the first seemed to affect only the **dimmer**, not the button action.
- Fixes that **did not** fully solve it alone: raising z-index, Radix overlay tweaks, `modal={false}`, replacing Radix with a plain portal, and shell `pointer-events` lock — the **decisive** fix was architectural (below).

---

## Root cause (confirmed)

**Multiple full-screen portals stacked.**

`WorkAreaContent` mounts **several drawer components** at once (active + **cached**). Each workflow form had mounted its own **`WorkflowFormDiscardDialog`** with local `showDiscardDialog` / `discardRequest` state. That produced **multiple** `createPortal(..., document.body)` trees and competing hit targets, even when only one dialog looked “open.”

Contributing factors (layering / focus) mattered less than **duplicate portal instances** from per-drawer mounts.

---

## Final architecture (current)

| Piece | Behavior |
|--------|----------|
| **Single dialog** | **`WorkflowFormDiscardDialog`** is rendered **once** in **`components/layout/app-layout.tsx`**, not inside each drawer entry file. |
| **Open state** | `open` ⇔ **`workflowFormNavigateAwayRequested !== null`** in `navigation-store`. No per-form dialog state. |
| **Dirty sync** | **`useWorkflowFormDirty(drawerId, isDirty)`** — only syncs `workflowFormDirty`; third-arg callback **removed**. |
| **Blocked navigation** | **`attemptWorkflowNavigate`** (and similar) sets **`setWorkflowFormNavigateAwayRequested(...)`**. Request stays in the store until the user acts or dismisses. |
| **In-app Cancel** | Forms call **`useNavigationStore.getState().setWorkflowFormNavigateAwayRequested(target)`** when the user cancels with a dirty form (same payload shape as navigate-away). |
| **Shell inert while open** | **`body[data-discard-confirm-open="true"]`**, **`discardConfirmOpen`** in store, **`gz-shell-block-pointer`** + CSS in **`app/globals.css`**, mobile scrim/sheet guards in **`work-area-mobile.tsx`**. |

### Store (excerpt)

```ts
// stores/navigation-store.ts
workflowFormNavigateAwayRequested: WorkflowNavigateAwayRequest | null;
clearWorkflowFormNavigateAwayRequested: () => void;
discardConfirmOpen: boolean;
setDiscardConfirmOpen: (v: boolean) => void;
```

### Hook (current)

```ts
// hooks/useWorkflowFormDirty.ts — dirty sync only
export function useWorkflowFormDirty(drawerId: string, isDirty: boolean) {
  useEffect(() => {
    const matches = activeDrawer === drawerId && WORKFLOW_DRAWERS.has(drawerId);
    setWorkflowFormDirty(matches && isDirty);
    return () => setWorkflowFormDirty(false);
  }, [drawerId, isDirty, activeDrawer, setWorkflowFormDirty]);
}
```

### Discard dialog (current)

- **`components/workflow/WorkflowFormDiscardDialog.tsx`**: subscribes to **`workflowFormNavigateAwayRequested`**, `createPortal` to **`document.body`**, **`z-[120]`**, plain backdrop + panel (no Radix `Dialog` for this flow).
- **`components/layout/app-layout.tsx`**: `<WorkflowFormDiscardDialog />` next to desktop/mobile shell.

---

## Z-index reference (unchanged, for context)

| Surface | Approx. z-index |
|---------|-----------------|
| Map | `z-0` |
| Mobile scrim / sheet | `z-40` / `z-50` |
| Desktop work area | `z-[90]` |
| Sidebar | `z-[100]` |
| Other Radix dialogs | `z-[110]` / `z-[111]` (`components/ui/dialog.tsx`) |
| Discard confirm | `z-[120]` |

---

## Historical attempts (chronological)

1. Raise dialog above work area / sidebar (`z-[110]` / `z-[111]`); Select dropdown to `z-[200]`.
2. Custom overlay / avoid Radix `RemoveScroll` on discard path.
3. `modal={false}` — **reverted** (regressions).
4. Plain portal + no Radix for discard — helped but **double-tap persisted** until (5).
5. **`gz-shell-block-pointer`** + **`data-discard-confirm-open`** — reduces shell stealing taps; not sufficient alone.
6. **Single global `WorkflowFormDiscardDialog` + store-held request** — **resolved** the issue.

---

## Files to read (maintenance)

| File | Purpose |
|------|---------|
| `components/layout/app-layout.tsx` | Mounts the single `WorkflowFormDiscardDialog` |
| `components/workflow/WorkflowFormDiscardDialog.tsx` | Confirm UI |
| `hooks/useWorkflowFormDirty.ts` | Dirty flag only |
| `stores/navigation-store.ts` | `WORKFLOW_DRAWERS`, `workflowFormNavigateAwayRequested`, `discardConfirmOpen` |
| `app/globals.css` | `body[data-discard-confirm-open="true"] .gz-shell-block-pointer` |
| `components/work-area/work-area-mobile.tsx` | Scrim / sheet; guards when `discardConfirmOpen` |
| `docs/audits/drawer-navigation.md` | Product rules for dirty navigation |

**Do not** add another `WorkflowFormDiscardDialog` inside `app/drawers/**` — see `.cursor/rules/workflow-drawers.mdc`.

---

## i18n (`workflowForm.*`)

- `discardTitle`, `discardDescription`, `keep`, `discard` — see `lib/i18n/messages.ts`.

---

## Optional forensic notes (if regressions)

- Temporarily log **capture**-phase `pointerdown` / `click` on `window` and compare first vs second tap targets.
- Confirm only **one** discard portal node exists in the DOM when the confirm is open.

---

*Last updated after confirm UX verified fixed; architecture reflects the single-dialog + store-driven implementation.*
