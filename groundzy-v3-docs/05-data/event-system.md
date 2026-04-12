# Event System (history & activity)

Part of the **larger data model**—not the whole model. See [`data-model-overview.md`](./data-model-overview.md), [`entities.md`](./entities.md), other companion docs under `05-data/`, and foundation docs under `00-foundation/` when present.

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

**Weather “timeline”** is **not** this system—it is forecast API data (e.g. `/api/weather/timeline`).

---

## 2. Why this is fragmented

- **Three representations** of tree-care events: embedded arrays, `tree_events`, and `work_items` rows tied by `source.historyEntryId`.
- **Workflow** adds **four large documents** plus **mirrors** in `work_items`.
- **Merge** happens in **`TreeHistoryView`** (client) when flags on—**not** a single server-side event stream.

---

## 3. Unified Event model (v3 — decided)

**Status:** Groundzy v3 is **event-first** per [`../00-foundation/principles.md`](../00-foundation/principles.md). This section is the **system definition** for the append-only **Groundzy event** stream (e.g. `groundzy_events/{eventId}`).

**Practical shape** (fields align with `lib/groundzy/events/schema/system.ts`; may evolve):

| Field | Role |
|-------|------|
| `schemaVersion` | Forward compatibility |
| `id` | Unique event id |
| `type` | Discriminant (`GroundzyEventType` and future extensions) |
| `organizationId` | Tenant scope |
| `actorUserId` | Who caused the event (see **system actors** under derived events) |
| `correlationId` | Request or chain id |
| `causationEventId` | Optional link to a prior event in the same chain |
| `subject` | What entity the event is about (`tree`, `property`, `job`, …) |
| `payload` | Type-specific body |
| `createdAtMs` | Server-accepted time |

**Today’s canonical `GroundzyEventType` values** in code are **workflow** and **tree** lifecycle events only (see `lib/groundzy/events/schema/system.ts`). Weather and intelligence are **not** yet first-class types in that union—[§4](#4-derived--computed-events-step-3) describes how **derived / computed** events fit when added.

### 3.1 Projections and commercial workflow

| Concept | Definition |
|---------|------------|
| **Domain / source event** | Append-only record of a **business fact** (user action, API command, approved write path). |
| **Derived event** | Append-only record of **system interpretation** (rules, projections, schedulers)—see §4. |
| **Projection** | Activity UI, public summaries, CRM dashboards, “upcoming work,” and **WorkItem-shaped** rows are **views** over events ± workflow document state—not separate authoritative histories. |
| **Commercial workflow** | Request/Quote/Job/Invoice remain **documents** for billing and lifecycle; **transitions** and meaningful changes are reflected in the **event** stream. |

**Legacy doc:** `docs/architecture/work-item-system-architecture.md` (in the app repo) claimed WorkItem as **system of record**—that is **superseded** for v3. See banner at top of that file; v3 does **not** treat WorkItem as canonical.

Event-first does **not** imply heavyweight streaming infrastructure: **Firestore (or equivalent) with structured event documents and projections** is sufficient unless product scale demands more.

---

## 4. Derived / computed events (Step 3)

**Definition:** A **derived** (or **computed**) event is an append-only record of **what the system concluded**, **detected**, or **decided to surface**—not a direct encoding of a single user gesture or raw provider webhook. It is produced by **evaluation** (rules, projections, schedulers) from:

- **Domain state** (trees, workflow, properties), and/or
- **Inputs** that are not themselves Groundzy mutations (e.g. forecast refresh, time passing, batch jobs).

This complements **source / domain events** (e.g. `tree.service_logged`, `workflow.job_created_from_quote`), which record **facts of record** in the business. Derived events record **interpretation** and support “why did I see this?” and analytics.

### 4.1 Contrast: domain vs derived

| | **Domain / source events** | **Derived / computed events** |
|--|-----------------------------|--------------------------------|
| **Typical producer** | User action, API command, approved write path | Rules engine, intelligence pipeline, scheduled evaluator |
| **Typical content** | “Service was logged,” “Quote was created” | “Tree risk elevated given forecast × health,” “Notification created for property X” |
| **Audit story** | What happened in the product | What the system inferred or triggered |

Naming in catalogs often uses prefixes such as `intelligence.*`, `notification.*`, or `weather.evaluation.*` (exact names TBD with the canonical event catalog).

### 4.2 Relationship to inputs and causality

- **Causation:** Derived events should set **`causationEventId`** when a specific **domain event** or **ingestion run** triggered evaluation (e.g. `weather.forecast.updated` → `intelligence.tree_risk.detected`).
- **Correlation:** Use **`correlationId`** to group batches (same forecast fetch, same nightly job).
- **Subject:** Point at the same **`subject`** model as domain events (e.g. `tree`, `property`) so projections and UI stay consistent.

### 4.3 Actors

When no human clicked the action, **`actorUserId`** should follow a documented policy (e.g. dedicated **system** or **service** user id per environment). This keeps streams queryable and avoids fake human ids.

### 4.4 Idempotency and dedupe

Derived emissions must align with **dedupe** rules from the intelligence layer (see [`intelligence-rules.md`](./intelligence-rules.md)): the same rule + tree + evaluation window should not create unbounded duplicate events. Prefer **stable dedupe keys** in payload or idempotent writers.

### 4.5 Illustrative examples (names not final)

| Type (illustrative) | Produced when | Typical `subject` |
|---------------------|----------------|-------------------|
| `intelligence.rule_fired` | Rule engine evaluates true | `property` / `tree` |
| `intelligence.tree_risk.detected` | Combined weather × tree threshold | `tree` |
| `notification.created` | Delivery layer commits an in-app / email / SMS item | `user` or `organization` |
| `workflow.job_suggested` | System recommends a job (optional automation) | `property` / `tree` |

Storage may use the **same** `groundzy_events` collection with extended `type` enums, or a **parallel** stream—implementation decision. Semantics remain: **append-only**, **typed**, **auditable**.

### 4.6 Related documents

- [`intelligence-rules.md`](./intelligence-rules.md) — rule families and severities that feed derived triggers.
- [`../07-systems/alert-notification-system.md`](../07-systems/alert-notification-system.md) — evaluation → trigger → delivery → workflow (when present).

---

## 5. Deprecation targets (legacy → v3)

- **`workflow_*` mirror kinds** in `work_items` — legacy; v3 **derives** operational views from events + CRM documents.
- **Dual-write** (embedded history + `tree_events` + `work_items`) — replace with **single write path** to canonical events + optional materialized projections.

---

## 6. Open decisions

- Canonical names and JSON schemas for `intelligence.*`, `weather.*` ingestion, and `notification.*`.
- Whether all derived events share **one** Firestore collection with domain events or a **separate** collection with shared envelope shape.
- Retention and PII for notification-related derived events.

---

## Related

- [`intelligence-rules.md`](./intelligence-rules.md)
- [`../07-systems/alert-notification-system.md`](../07-systems/alert-notification-system.md)
- [`data-flows.md`](./data-flows.md)
- [`../00-foundation/rebuild-audit-history-and-uiux.md`](../00-foundation/rebuild-audit-history-and-uiux.md)
