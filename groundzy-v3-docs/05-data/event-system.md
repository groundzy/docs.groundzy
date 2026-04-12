# Event System (history & activity)

Part of the **larger data model**—not the whole model. See [`data-model-overview.md`](./data-model-overview.md) and [`entities.md`](./entities.md).

---

## 1. What exists today (fragmentation)

| Mechanism | Storage | Purpose |
|-----------|---------|---------|
| **Embedded `TreeHistory`** | `trees/{id}.history` arrays | Service, inspection, note, measurement, access entries (`types/tree.ts`) |
| **TreeEvent subcollection** | `tree_events/{treeId}/events/{eventId}` | Phase 4 append-only; `data` holds full `HistoryEntry` |
| **WorkItem** (non-mirror kinds) | `work_items` | Operational index: service, inspection, note, measurement, zone_service — **dual-written** from tree history when enabled |
| **WorkItem** (`workflow_*` kinds) | `work_items` | **LEGACY_MIRROR** of Request/Quote/Job/Invoice (`types/work-item.ts`) |
| **CRM status fields** | `requests`, `quotes`, `jobs`, `invoices` | Status enums — **not** a separate audit-log collection |
| **Public activity** | `tree_public_summaries`, `Tree.activity` types | Redacted **PublicActivityEntry** — projection for public views |
| **User preferences** | `users.preferences` | `workItemsActivityTimelineEnabled` merges work_items + history in **UI** only |

**Weather “timeline”** is **not** this system—it is forecast API data (`/api/weather/timeline`).

---

## 2. Why this is fragmented

- **Three representations** of tree-care events: embedded arrays, `tree_events`, and `work_items` rows tied by `source.historyEntryId`.
- **Workflow** adds **four large documents** plus **mirrors** in `work_items`.
- **Merge** happens in **`TreeHistoryView`** (client) when flags on—**not** a single server-side event stream.

---

## 3. Unified Event model (v3 — decided)

**Status:** Groundzy v3 is **event-first** per [`../00-foundation/principles.md`](../00-foundation/principles.md). This section is the **system definition**, not an optional recommendation.

**Practical shape** (fields may evolve in implementation):

| Field | Role |
|-------|------|
| **id** | Stable event identifier |
| **type** | Typed event name (e.g. `tree.created`, `job.scheduled`, `workflow.quote_converted`) |
| **subject** | What the event is about: `entityType` + `entityId` (tree, zone, property, job, …) |
| **timestamp** | When it occurred (and ordering for timelines) |
| **payload** | Typed payload for the event `type` (measurements, notes, transition metadata, etc.) |
| **scope** | Org/user context as required for rules and privacy |

| Concept | Definition |
|---------|------------|
| **Event** | The **only** durable record of an action. Append-only; no competing “truth” for the same fact. |
| **Projection** | Activity UI, public summaries, CRM dashboards, “upcoming work,” and **WorkItem-shaped** rows are **views** over Events ± workflow document state—not separate authoritative histories. |
| **Commercial workflow** | Request/Quote/Job/Invoice remain **documents** for billing and lifecycle; **transitions** and meaningful changes are reflected in the **Event** stream. |

**Legacy doc:** `docs/architecture/work-item-system-architecture.md` claimed WorkItem as **system of record**—that is **superseded** for v3. See banner at top of that file; v3 does **not** treat WorkItem as canonical.

Event-first does **not** imply heavyweight streaming infrastructure: **Firestore (or equivalent) with structured event documents and projections** is sufficient unless product scale demands more.

---

## 4. Deprecation targets (legacy → v3)

- **`workflow_*` mirror kinds** in `work_items` — legacy; v3 **derives** operational views from Events + CRM documents.
- **Dual-write** (embedded history + `tree_events` + `work_items`) — replace with **single write path** to canonical Events + optional materialized projections.

---

## Related

- [`Groundzy v3/00-foundation/rebuild-audit-history-and-uiux.md`](../00-foundation/rebuild-audit-history-and-uiux.md)
- [`data-flows.md`](./data-flows.md)
