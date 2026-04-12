# Event & history system (cross-feature)

**Nature:** Shared concern across **trees**, **dashboard**, **work_items**, **workflow mirrors**, and **public** views—not a single “feature.”

---

## What exists today (fragmented)

| Layer | Mechanism | Consumers |
|-------|-----------|------------|
| **Embedded tree history** | `trees.history` arrays (`TreeHistory`, `HistoryEntry`) | Tree Activity, forms |
| **Tree events** | `tree_events/{treeId}/events/{eventId}` | Hydration / migration path (`lib/firebase/firestore.ts`) |
| **Work items** | `work_items` — operational kinds + **`workflow_*` LEGACY_MIRROR** | Dashboard upcoming, Activity tab merge, zones |
| **CRM status** | Document-level `status` on Request/Quote/Job/Invoice | Workflow UI |
| **Public activity** | `PublicActivityEntry`, `tree_public_summaries` | Restricted/public views |
| **User prefs** | `workItemsActivityTimelineEnabled`, dual-write flags | UI merge behavior without schema unity |

**Evidence:** `types/tree.ts`, `types/work-item.ts`, `lib/firebase/work-items.ts`, `Groundzy v3/05-data/event-system.md`, `Groundzy v3/00-foundation/rebuild-audit-history-and-uiux.md`.

---

## Duplication

- Same **care event** can be represented in **embedded history**, **tree_events**, and **work_items** (dual-write).
- **Workflow** entities and **`workflow_*` work items** duplicate commercial state.
- **Merge logic** in **`TreeHistoryView`** dedupes by `source.historyEntryId`—presentation-layer fix, not a single source of truth.

---

## Fragmentation

- No **one** append-only **Event** type enforced across domains (conceptual model exists in v3 product docs only).
- **Weather “timeline”** (API) vs **activity timeline** (UI)—naming collision only (`Groundzy v3/06-features/weather.md`).

---

## Missing abstraction layers

| Gap | Impact |
|-----|--------|
| **Unified event contract** | Features each know too much about storage paths. |
| **Event projection API** | Dashboard and Activity reimplement merge rules. |
| **Audit trail for workflow** | Status on documents; no separate immutable transition log in surveyed types. |

---

## v3 foundation direction

- **One conceptual Event stream** with **typed payloads** and **subject** (tree, zone, job, …).
- **Materialized views** (Activity, public feed) = queries/projections, not second writes.
- **Deprecate** dual-write and **`workflow_*` mirrors** on a planned path (`types/work-item.ts` already flags legacy).

---

## Related

- `Groundzy v3/05-data/event-system.md`
- `lib/workflow/work-item-adapters.ts`
