# Tree activity — Groundzy v3 migration status

This document summarizes **what is implemented** for tree activity moving onto the v3 append pipeline (`groundzy_events` → projections → `tree_timeline_entries`) and **what remains** on legacy paths. Last updated from the app repo as of the migration work that landed note/service/measurement/inspection events, activity consolidation, entry-form reliability fixes, and remaining menu cutovers.

---

## Done

### Event types (append + validation + enrichment + projection)

| Event | Purpose (repo terms) |
|-------|------------------------|
| `tree.note_added` | Notes (includes UI “Reminder” when saved as a `NoteRecord`). |
| `tree.service_logged` | Services — main **Service** row, **Treatment** (corrective service record), and **Schedule** branches that produce a `ServiceRecord`. |
| `tree.measurement_recorded` | Measurements — main **Measurement** row and **Schedule → measurement** placeholder rows. |
| `tree.inspection_recorded` | Inspections — UI **Assessment** standalone form and **Schedule → assessment** (scheduled health inspection). |
| `tree.note_revised` / `tree.service_revised` / `tree.measurement_revised` / `tree.inspection_revised` | Edit existing v3 timeline row (same payload shapes as add; server merges into existing `tree_timeline_entries` doc, preserves `metadata.createdAt`). |
| `tree.timeline_entry_removed` | Delete a v3 timeline row (discriminated by `entryKind` + id fields). |

- **Canonical read path for migrated kinds:** `lib/firebase/tree-timeline-activity.ts` — `fetchTreeTimelineProjection()` (single `tree_timeline_entries` query) + `resolveTreeHistoryReadModel()` / `mergeTimelineProjectionIntoBaseHistory()` (timeline order primary; embedded `trees.history` fills ids not in the projection). **`tree_events` is not read by the app.**
- **Consumers:** `getTree()` / `getTreeHistory()` in `lib/firebase/firestore.ts` merge embedded history + timeline only.

### Entry form — single-tree **new** row (`app/drawers/entry-form/index.tsx`)

- **v3 append** for **edit** when the row has `metadata.version === "v3-timeline"`: `reviseTree*ViaGroundzyEvent` → the corresponding `tree.*_revised` event.
- **v3 append** for new single-tree saves (not edit, not bulk) where applicable:
  - Primary **Service** (not treatment sub-flow) → `addTreeServiceViaGroundzyEvent`
  - **Note** / **Reminder** → `addTreeNoteViaGroundzyEvent`
  - **Measurement** → `addTreeMeasurementViaGroundzyEvent`
  - **Assessment** (`entryType === "inspection"`) → `addTreeInspectionViaGroundzyEvent`
  - **Treatment** (`recordTypeOption === "treatment"`) → `addTreeServiceViaGroundzyEvent`
  - **Schedule** → routes by record shape: service → `addTreeServiceViaGroundzyEvent`, measurement → `addTreeMeasurementViaGroundzyEvent`, inspection → `addTreeInspectionViaGroundzyEvent`; unknown shape falls back to `addTreeHistoryEntry` (safety net).

### Add tree form — non-drawer history (`components/trees/add-tree-form.tsx`)

- **Edit tree — inline Activity:** `persistInlineAddTreeHistoryEntry` (same v3 vs legacy rules as entry-form); delete uses `deleteHistoryEntryForAddTreeForm` (v3 remove vs legacy delete).
- **Edit tree — measurement/health change:** `appendMeasurementAndInspectionSeedV3` (v3 measurement + inspection append only; no `addTreeHistoryEntry` for those seeds).
- **Create tree — measurement/health seed:** same `appendMeasurementAndInspectionSeedV3`.
- **Create tree — inline Activity** before first save: **`appendCreateFlowHistoryEntriesV3`** after `createTree` (no embedded activity blob on create). See `lib/groundzy/append-create-flow-tree-history-v3.ts`.

### Entry form — **bulk** add (`params.bulkTreeIds`)

- **`bulkAppendNewTreeActivity`** in `lib/groundzy/client/bulk-append-tree-activity.ts`: for each target tree id, `getTree` → `withEntryForTree` (GZ-TIN + `treeId`) → same v3 vs legacy rules as single-tree new entry.
- **One event per tree** (unchanged); idempotency remains per `(organizationId, idempotencyKey)` where keys come from existing append clients (`stableAppendIdempotencyKey(entry.id, treeId)`).
- **Partial failure:** per-tree try/catch; toast reports successes vs failures; query invalidation only for **succeeded** tree ids.
- **Still legacy per tree** when the same shape would use `addTreeHistoryEntry` in single-tree mode (e.g. Schedule safety-net unknown shape).

### UX / reliability

- Stable draft entry ids in `components/trees/add-history-entry-form.tsx` (`nextNewEntryId()` / `crypto.randomUUID`) so double-submit does not create duplicate idempotency keys.
- Save loading state and in-flight guard on `entry-form` (disable buttons, “Saving…”).

### Activity UI consolidation

- **Tree Activity** tab no longer merges `work_items` into `TreeHistoryView` (`TreeActivityTab` removed `useWorkItemsForTree` / preference flag for this surface).
- Preference `workItemsActivityTimelineEnabled` is **deprecated for tree Activity** (may still affect zone work index / profile toggles elsewhere — see code comments).

### v3 edit / delete (view tree + entry form)

- **Edit:** opens entry form as before; save uses revise events for v3 rows (`app/drawers/entry-form/index.tsx`).
- **Delete:** confirm uses `removeTreeTimelineEntryViaGroundzyEvent` → `tree.timeline_entry_removed` for v3 rows (`app/drawers/view-tree/index.tsx`). Legacy rows still use `deleteTreeHistoryEntry`.

### Tests (representative)

- Vitest coverage for validation, enrichment, projection handlers, append clients, idempotency/duplicate responses, and `tree-timeline-activity` merge behavior (see `lib/groundzy/**/*.test.ts`, `lib/firebase/tree-timeline-activity.test.ts`).

### Key files (reference)

- Schemas: `lib/groundzy/events/schema/tree.ts`, `lib/groundzy/events/schema/system.ts`
- Validate: `lib/groundzy/events/validate.ts`
- Append server: `lib/groundzy/server/append-event.ts`
- Policy: `lib/groundzy/policy/can-append-event.ts`
- Projections: `lib/groundzy/projections/handlers/tree/*.ts`, `lib/groundzy/projections/handlers/index.ts`
- Timeline read/merge: `lib/firebase/tree-timeline-activity.ts`
- Clients: `lib/groundzy/client/add-tree-*-via-event.ts`, `revise-tree-*-via-event.ts`, `remove-tree-timeline-entry-via-event.ts`, `bulk-append-tree-activity.ts`
- Projection delete effect: `lib/groundzy/projections/types.ts`, `lib/groundzy/projections/apply.ts`
- Firestore tree load: `lib/firebase/firestore.ts` (`getTree`, `getTreeHistory`)

---

## Yet to do

### High — product-facing gaps

1. **Edit existing row** (`updateTreeHistoryEntry` in `entry-form`)  
   Still **legacy** for rows that are **not** v3 timeline (`metadata.version !== "v3-timeline"`).

2. **Non–entry-form flows**  
   e.g. **`components/trees/add-tree-form.tsx`** and any other callers that add history without the append API — still **legacy** where not explicitly migrated.

### Medium — optional cleanup

3. **Legacy embedded helpers** (`addTreeHistoryEntry` / `updateTreeHistoryEntry` / `deleteTreeHistoryEntry`)  
   Still used for **non–v3-timeline** rows and schedule/edge fallbacks; they update **embedded `trees.history` only** (no `tree_events`). Further reduction is optional and caller-by-caller.

4. **Profile / copy**  
   “Unified tree activity” preference text may still imply tree merging; align copy with current behavior if product wants.

### Lower — completeness

5. **Access / sharing history** (`accessHistory`)  
   Stored on **embedded `trees.history` only** (not in `tree_timeline_entries`); intentional for this product version.

6. **Schedule safety-net branch**  
    `addTreeHistoryEntry(tree.id, entry)` when schedule produces an unrecognized `HistoryEntry` — rare; optional logging if desired.

7. **Ordering**  
    Merge does not sort; UI sorts by date. If strict global ordering vs `sortKey` on docs is required everywhere, confirm in list views that consume `tree.history` only.

---

## Verification commands (dev)

```bash
npm run test -- --run lib/groundzy lib/firebase/tree-timeline-activity.test.ts
npx tsc --noEmit
```

(Adjust test command to match your `package.json` scripts.)

---

## Terminology cheat sheet

| UI label | Repo / type | v3 event (when applicable) |
|----------|-------------|----------------------------|
| Assessment | `InspectionRecord`, `entryType` inspection | `tree.inspection_recorded` |
| Service / Treatment / Schedule→service | `ServiceRecord` | `tree.service_logged` |
| Measurement / Schedule→measurement | `MeasurementRecord` | `tree.measurement_recorded` |
| Note / Reminder | `NoteRecord` | `tree.note_added` |

---

*Cleanup pass (v3): `tree_events` removed from app paths; types `TreeEvent` / `TreeEventType` removed from `types/tree.ts`; obsolete `listTreeTimeline*` wrappers removed from `firestore.ts`; Phase 4 `migrate-tree-events` script archived under `scripts/archive/`.*

*This file is a handoff snapshot; update it when you ship further optional legacy reduction.*
