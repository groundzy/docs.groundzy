# Groundzy work domain — master plan

This document **synthesizes** the product vision from the service taxonomy / ServiceType / WorkItem discussions with **what is already implemented** and **what remains**. It does not replace the detailed migration spec; it orients teams around one spine and one roadmap.

**Related documents**

| Document | Role |
|----------|------|
| [Work domain master plan (Cursor)](../../.cursor/plans/groundzy-work-domain-master-plan.plan.md) | **Trackable todos** for Phases A–F + decision checkpoint |
| [WorkItem migration plan (Cursor)](../../.cursor/plans/workitem_migration_plan_8721b13b.plan.md) | Code-grounded migration: dual-write, Activity tab, `workflow_*` legacy mirrors, Job as container — **implementation source of truth** |
| [Tiers vs workflow audit](./tiers-workflow-vs-record-keeping-audit.md) | Tier vs record-keeping split |
| [Tree service taxonomy](../reference/tree-service-taxonomy.md) | Human-readable master list of services by category |
| [ServiceType model spec](../reference/service-type-model-spec.md) | Proposed **catalog** enums and `ServiceTypeDefinition` |
| [Work item system architecture](./work-item-system-architecture.md) | Aspirational CRM-centric WorkItem fields (e.g. `quoted` / `approved` statuses, `serviceTypeId`) — **conceptual target**, see §3 for alignment with shipped types |

---

## 1. Spine (north star)

> **Everything actionable is a WorkItem (or flows through one). Everything historical is a Record. Everything billable or schedulable should eventually reference WorkItems.**

- **ServiceType** (catalog) = *what kind of work* — stable taxonomy, defaults for billing/targeting/history flavor.
- **WorkItem** (Firestore `work_items/{id}`) = *one unit of schedulable / reportable work* — the operational system of record for queries, Activity, and (over time) CRM alignment.
- **Tree history** (`serviceHistory`, `inspectionHistory`, etc. + `tree_events`) = *output log* — chronological, user-readable; not where commercial or multi-step execution state should live alone.
- **Request → quote → job → invoice** = *commercial / project containers* — should reference work units instead of re-embedding service logic long term.

---

## 2. What the chat produced (documentation)

1. **Master taxonomy** — Fifteen categories from planting through consulting; maps product language to future `ServiceCategory` / labels.
2. **ServiceType spec** — `ServiceTypeId`, `ServiceTypeDefinition`, billing/execution defaults, `writesHistoryAs` to keep history classification consistent.
3. **WorkItem architecture (aspirational)** — Richer lifecycle (`draft` → … → `completed` with `quoted` / `approved`), `targets[]`, `WorkItemEstimate`, explicit `serviceTypeId`, RecurringPlan as generator.

These docs are **intentional targets** for UX and data modeling; they are not yet all reflected in TypeScript or Firestore.

---

## 3. Current implementation (repo reality) — snapshot

Decisions **already locked** in the migration plan (do not casually overturn):

- **`Job`** (`jobs/{id}`) remains a **container** (scheduling, client, pricing context). WorkItem mirrors jobs in v1; **child WorkItems** under a job are a **v2+** direction (`parentWorkItemId` reserved).
- **`workflow_request` | `workflow_quote` | `workflow_job` | `workflow_invoice`** are **LEGACY_MIRROR** kinds on WorkItem — mirrors of existing CRM docs, not the final domain taxonomy.
- **Dual-write** from tree history, zone services, and workflow creates/updates **is shipped** (see migration plan todos). Activity merges WorkItems + legacy history where implemented.
- **Operational WorkItem** today uses **`kind`** + **`stage`** + **`details`**, **`primaryEntity`**, **`treeIds`**, **`workflow`**, **`source`** — see [`types/work-item.ts`](../../types/work-item.ts). This differs from the aspirational doc’s `serviceTypeId`-first shape and separate commercial status enum.

**Gap:** There is **no** `types/service-types.ts` / `SERVICE_CATALOG` in repo yet. **No** `workItemIds[]` on `Request` / `Quote` / `Job` / `Invoice` types yet — line items and `treeIds` on those documents remain the integration surface today.

---

## 4. Strategic pillars

| Pillar | Intent |
|--------|--------|
| **P1 — Taxonomy** | Introduce **ServiceType** as versioned catalog (code-first, optional Firestore later) so labels, defaults, and “what history flavor” are consistent. |
| **P2 — WorkItem stays canonical for ops** | Continue to treat **`work_items`** as the queryable layer for trees/zones/org; expand fields only when product needs them (assignment, estimates, explicit targets). |
| **P3 — History stays a log** | On completion, **summarize** into tree history / events; avoid duplicating full CRM state in `HistoryEntry`. |
| **P4 — CRM convergence** | Move from “mirrored aggregate row per doc” toward **linking** quotes/jobs/invoices to **specific WorkItem ids** for line-level work — **without** removing Job as container. |
| **P5 — Zones** | Prefer **WorkItems** with zone `primaryEntity` over a permanent parallel **`zone_services`** model (transitional dual path acceptable). |
| **P6 — Recurrence** | **RecurringPlan** (or equivalent) **generates** WorkItems; it is not a WorkItem. |

---

## 5. Roadmap (phased)

Phases **build on** the completed migration-plan phases (dual-write, Activity, capabilities). Numbers here are **product phases**, not a repeat of file-level migration §6.

### Phase A — ServiceType catalog (types + data, low UI)

- Add **`types/service-types.ts`** (and optionally **`lib/service-catalog.ts`**) per [service-type model spec](../reference/service-type-model-spec.md).
- Wire **i18n labels** for `ServiceTypeId` / categories when surfaced in UI.
- **Bridge field (decision):** add optional **`serviceTypeId?: ServiceTypeId`** on WorkItem `details` or as optional top-level field once validated — or maintain a **mapping** from `(kind + details.serviceName)` → `ServiceTypeId` in adapters until backfill exists.

**Exit criteria:** Dev can list/filter catalog entries; at least one UI path shows a resolved label (e.g. service history or WorkItem detail).

### Phase B — Enrich WorkItem where product needs it

Pick fields from [work-item-system-architecture.md](./work-item-system-architecture.md) **incrementally**:

- **Assignment** (`assignedUserIds`, crew label) if scheduling UX requires it.
- **Estimate** block if quoting/scheduling shares the same row.
- **Explicit `targets[]`** only if `primaryEntity` + `treeIds`/`zoneIds` becomes limiting for multi-entity jobs.

Avoid copying the entire aspirational interface in one PR — **extend** current [`WorkItem`](../../types/work-item.ts) to stay compatible with existing Firestore documents and rules.

**Exit criteria:** New fields are optional, indexed only if queried, and documented in [firestore-collections](../reference/firestore-collections.md).

### Phase C — CRM: `workItemIds` on workflow documents

- Add **`workItemIds?: string[]`** (or per-line references) to **Request**, **Quote**, **Job**, **Invoice** when product is ready to price/schedule **the same rows** across stages.
- Keep **Job document** authoritative for job-level status until a deliberate cutover; WorkItems link **into** the container.
- Mirrored **`workflow_*` WorkItems** may remain for timeline/query until reads migrate.

**Exit criteria:** At least one vertical (e.g. job → invoice) can show linked WorkItems without duplicating service definitions in line item text.

**Implemented:** Types + create/convert paths set `workItemIds` via [`lib/workflow/workflow-document-work-item-ids.ts`](../../lib/workflow/workflow-document-work-item-ids.ts); **View invoice** loads linked rows with [`useWorkItemsByIds`](../../hooks/useWorkItems.ts) and offers **Open job**.

### Phase D — Recurring care

- Implement **`RecurringPlan`** collection (or schedule on org/property) that **creates** WorkItems on `nextRunAt` / cron / client-triggered generation.
- Align `serviceTypeId` and targets with catalog.

**Exit criteria:** A recurring inspection creates a schedulable WorkItem per run, visible on tree/Activity.

**Implemented:** [`types/recurring-plan.ts`](../../types/recurring-plan.ts), [`lib/firebase/recurring-plans.ts`](../../lib/firebase/recurring-plans.ts), [`TreeRecurringPlansSection`](../../app/drawers/view-tree/components/TreeRecurringPlansSection.tsx) on **Operations** (add yearly plan + schedule next → new `work_items` row, Activity when merged).

### Phase E — Zone services consolidation

- **Preferred:** new zone-scoped work uses **WorkItem** with `primaryEntity: { type: "zone" }`; reduce new writes to `zone_services` over time.
- **Transitional:** keep `zone_services` readers working; optional adapter that **materializes** WorkItem if missing.

**Exit criteria:** Zone drawer and map views can rely on WorkItems for “what’s scheduled here” with parity checks.

**Implemented:** `listWorkItems` filters by `zoneIds` array-contains; [`useWorkItemsForZone`](../../hooks/useWorkItems.ts); zone drawer [`ViewZoneWorkItemsIndexSection`](../../app/drawers/view-zone/ViewZoneWorkItemsIndexSection.tsx) (shown when `workItemsActivityTimelineEnabled`); composite index `organizationId` + `isDeleted` + `zoneIds` (array-contains) + `updatedAt`.

### Phase F — Dashboard & upcoming

- **Today:** upcoming items are derived largely from embedded **history** + scheduled service/inspection records.
- **Target:** optional query path from **WorkItem** (`stage`, `schedule`) with same UX guarantees as [dashboard-upcoming](../../lib/utils/dashboard-upcoming.ts) — only after indexes and parity validation.

**Implemented:** User pref `workItemsDashboardUpcomingEnabled`; [`listUpcomingScheduledWorkItemsForOrg`](../../lib/firebase/work-items.ts) + [`useScheduledWorkItemsForDashboard`](../../hooks/useWorkItems.ts); [`mergeDashboardUpcomingWithWorkItems`](../../lib/utils/dashboard-upcoming.ts) (dedupe by history entry, `serviceTypeId` / catalog labels, `zone_service` when `treeIds` present); dashboard loading waits for work-items query when merge is on; composite index `organizationId` + `isDeleted` + `stage` + `schedule.start`; tests in [`dashboard-upcoming.test.ts`](../../lib/utils/dashboard-upcoming.test.ts).

---

## 6. Decisions to lock before large CRM refactors

1. **Single vs multiple WorkItems per quote line** — one WorkItem per priced line vs one aggregate per quote.
2. **Commercial status vs `stage`** — whether `quoted` / `approved` live on WorkItem, on Job/Quote only, or as a **facet** separate from field execution `stage`.
3. **ServiceType backfill** — migrate historical `details` strings to `serviceTypeId` vs lazy resolution at read time.
4. **When to thin `workflow_*` kinds** — only after `workItemIds` + reads are trusted everywhere needed.

---

## 7. Risk register (short)

| Risk | Mitigation |
|------|------------|
| Two mental models (kind vs serviceTypeId) | Adapters + single resolver function; document in `work-item-adapters` tests |
| Firestore index / rules churn | Optional fields + phased indexes; update rules with each phase |
| UX regression on Activity | Keep merge + dedupe tests; feature-flag risky timeline changes |
| Over-large WorkItem document | Keep heavy blobs in `details` or subcollections later |

---

## 8. Summary

The **chat** established **taxonomy + ServiceType catalog + WorkItem-as-spine** as the long-term shape. The **repo** has already executed the **hard foundation**: **WorkItem collection**, **dual-write**, **legacy workflow mirrors**, and **Job-as-container**. The **master plan** is to layer **ServiceType**, enrich **WorkItem** only as needed, **link CRM documents to WorkItems**, add **RecurringPlan**, and **converge zone** work — without breaking the locked migration contracts already in production code paths.
