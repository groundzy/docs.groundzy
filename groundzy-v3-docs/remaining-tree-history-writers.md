# Groundzy v3 Remaining Tree History Writers (optional cleanup)

Legacy **embedded** mutators for tree activity. They **do not** write to `tree_events` (Phase 4 retired from app paths). Correctness for migrated kinds comes from **Groundzy v3 append / revise / remove** + `tree_timeline_entries`.

## Core helpers (`lib/firebase/firestore.ts`)

| Function | Behavior |
|----------|----------|
| `addTreeHistoryEntry` | Appends to embedded `trees.history` only; optional **work-item** sync. |
| `updateTreeHistoryEntry` | Updates embedded `trees.history` only; optional work-item sync. |
| `deleteTreeHistoryEntry` | Removes from embedded `trees.history` only; work-item soft-delete. |
| `addTreeAccessHistoryEntry` | Appends to **`accessHistory`** on the tree doc only (no timeline kind). |

## Call sites (app code)

| Location | Role |
|----------|------|
| `app/drawers/entry-form/index.tsx` | Fallback for non–v3-timeline edit; schedule safety-net; final else. |
| `lib/groundzy/client/bulk-append-tree-activity.ts` | Same parity fallback per tree. |
| `lib/groundzy/persist-inline-add-tree-history.ts` | Add-tree-form edit inline Activity; same rules. |
| `app/drawers/view-tree/index.tsx` | `deleteTreeHistoryEntry` when row is not v3 timeline-backed. |
| `app/drawers/contact-us.tsx`, `components/bulk/bulk-share-dialog.tsx` | `addTreeAccessHistoryEntry` after grant. |

## When these can be removed entirely

- No remaining callers **and** work-item sync policy decided for v3-only world **and** no product need for embedded-only legacy rows.

## Verification

```bash
npx tsc --noEmit
npx vitest run lib/groundzy lib/firebase/tree-timeline-activity.test.ts
```
