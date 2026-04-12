# Workflow system (cross-feature)

**Nature:** The **commercial spine** (Request → Quote → Job → Invoice) plus **supporting** pricing, scheduling, and **team workflow settings**—used by CRM drawers, map filters (jobs), dashboard, weather (`job_reschedule`), and **`work_items`** mirrors.

---

## Components (repository)

| Piece | Role |
|-------|------|
| **Entities** | `requests`, `quotes`, `jobs`, `invoices` (`types/*.ts`) |
| **Team defaults** | `teams.workflowSettings` — tax, currency, line templates (`getTeamWorkflowSettings`) |
| **Conversion** | Foreign keys + conversion helpers (`lib/workflow/*`, entity `convertedTo*` fields) |
| **Work items** | Mirrors + optional `workItemIds[]` on CRM docs |
| **Recurring plans** | `recurring_plans` → generated **work_items** / ops |

**Docs:** `docs/features/crm.md`, `docs/features/current-workflow-audit.md`, `workflow-upgrade-implementation.md`.

---

## Documentation map (legacy vs v3)

Legacy **`docs/features/crm.md`** bundles **clients/properties** with **requests → invoices** in one narrative. Groundzy v3 **splits** that story for clarity:

| Concern | v3 doc |
|---------|--------|
| Clients & properties | [`../06-features/crm.md`](../06-features/crm.md) |
| Request → Quote → Job → Invoice | [`../06-features/workflow.md`](../06-features/workflow.md) |

Cross-feature architecture: this file (`workflow-system.md`) and [`../05-data/relationships.md`](../05-data/relationships.md).

---

## Duplication

- **Job** document vs **`WorkItem`** (`workflow_job`) vs **tree history** when work is logged on trees—three lenses on related reality.
- **Line items** repeated shape across Quote/Job/Invoice types with mappers (`line-item-mappers`).

---

## Fragmentation

- **Navigation-heavy** history vs **relational** upgrades (audits describe both eras).
- **Workflow settings** live on **team** doc; **Pro** solo users use defaults elsewhere—potential **split brain** for tax/currency.
- **Request** vs **tree_access_request** — same word, different domain (`Groundzy v3/05-data/entities.md`).

---

## Missing abstraction layers

| Gap | Impact |
|-----|--------|
| **Workflow engine interface** | Drawers talk to Firestore shapes directly. |
| **Single “pipeline state machine”** | Status enums duplicated; sync between Request quoted vs Quote converted is historically fragile. |
| **Canonical link graph** | `workItemIds` optional; not all paths populate consistently. |

---

## v3 foundation direction

- **CRM documents remain** the billing source of truth unless replaced by a dedicated billing domain.
- **Work items** become **derived** or **merged** into Event + operational scheduling—**not** parallel mirrors long term.
- **WorkflowSettings** → explicit **org-level policy** object with one read path for Pro vs Team.

---

## Related

- `Groundzy v3/06-features/workflow.md`
- `Groundzy v3/05-data/relationships.md`
