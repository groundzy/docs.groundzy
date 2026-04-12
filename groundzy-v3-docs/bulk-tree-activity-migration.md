# Groundzy v3 Bulk Tree Activity Migration

## Current Legacy Bulk Flow (before this work)

- **Navigation:** `app/drawers/trees.tsx` passes `bulkTreeIds` (comma-separated tree ids) into the entry-form drawer params along with the primary `treeId`.
- **Form:** `app/drawers/entry-form/index.tsx` parses `bulkTreeIds`, merges the primary `tree.id` into the set, then **looped** `addTreeHistoryEntry(tid, entry)` for every id.
- **Record shape:** Built by `AddHistoryEntryForm` for the **primary** tree; the same object (including shared `entry.id`) was written to each tree’s embedded `trees.history` with Phase 4 `tree_events` dual-write and optional work-item sync — **no v3 append**.

## Implementation Summary

- **Module:** [`lib/groundzy/client/bulk-append-tree-activity.ts`](../lib/groundzy/client/bulk-append-tree-activity.ts)
  - `withEntryForTree(entry, tree)` — sets `treeId` and `gzTin` from each `Tree` (required for v3 payloads).
  - `appendNewTreeActivityForTree(...)` — one tree: `getTree` → aligned entry → same branching as single-tree new entry (v3 `addTree*ViaGroundzyEvent` vs `addTreeHistoryEntry`).
  - `bulkAppendNewTreeActivity(...)` — sequential loop, per-tree success/failure, counts `v3Count` / `legacyCount`.
- **Entry form:** [`app/drawers/entry-form/index.tsx`](../app/drawers/entry-form/index.tsx) — bulk branch calls `bulkAppendNewTreeActivity`, toasts on full / partial / total failure, early `return` on zero successes; invalidates `["tree", tid]` for **all succeeded** ids.

### Strategy

- **One append event per tree** (same as single-tree v3); no batch API change on the server.
- **Idempotency:** Unchanged — each client append still sends `stableAppendIdempotencyKey(entry.id, treeId)` (per-tree + shared entry id yields distinct keys across trees; retries for the same tree+entry still dedupe).
- **No dual-write on v3 paths** — append pipeline only; legacy path still uses `addTreeHistoryEntry` when the form would have used it for a single tree (e.g. Schedule unknown shape).

## Required Repo Changes (reference)

### File: `lib/groundzy/client/bulk-append-tree-activity.ts`

New file — see repository for full source.

### File: `app/drawers/entry-form/index.tsx`

- Import `bulkAppendNewTreeActivity`.
- Introduce `treeIdsToInvalidate` (default `[tree.id]`).
- Replace bulk `for` loop with `bulkAppendNewTreeActivity` + toasts.
- Replace single `invalidateQueries` for `tree.id` with `treeIdsToInvalidate.map(...)`.

## Tests to Add

### File: `lib/groundzy/client/bulk-append-tree-activity.test.ts`

See repository — covers:

- `withEntryForTree` GZ-TIN / `treeId` alignment.
- Bulk notes → v3 only (no `addTreeHistoryEntry`).
- Missing tree → failure row, no append.
- Schedule + non-service/measurement/inspection shape → legacy `addTreeHistoryEntry` with aligned tree fields.
- Primary service log → v3 service append.

## Legacy Bulk Paths to Retire

For **migrated shapes** (notes, primary service, measurement, inspection, treatment service, schedule service/measurement/inspection), bulk **no longer** uses:

- `addTreeHistoryEntry` alone in a dumb loop for those trees.

Those trees now go through **`addTree*ViaGroundzyEvent`** (same as single-tree).

## Remaining Legacy Paths

- **Bulk** still calls **`addTreeHistoryEntry`** per tree when the equivalent **single-tree** path would (Schedule safety-net unknown `HistoryEntry` shape, or any future edge case matching the `appendNewTreeActivityForTree` fallback).
- **Edit** non-v3 rows, **add-tree-form**, and other **non–entry-form** flows — unchanged.
- **`addTreeHistoryEntry` side effects** (embedded history, `tree_events`, work items) — still apply only when the legacy branch runs.

## Verification Steps

**Commands**

```bash
npx tsc --noEmit
npx vitest run lib/groundzy/client/bulk-append-tree-activity.test.ts --run
```

**UI**

1. From `trees` drawer, select multiple trees and add a **Note** (or Service / Measurement / Assessment / Treatment / Schedule with a migrated shape).
2. Save — expect success toast with correct tree count; Activity on each tree shows one new v3 row (merged timeline).
3. Simulate one invalid tree id (optional dev test) — expect partial error toast and only successful trees updated.
