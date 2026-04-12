# Workflow (requests → invoices)

## What it does

Commercial pipeline: **Request** (lead) → **Quote** → **Job** → **Invoice**, with statuses, line items, scheduling, and payments. **Recurring plans** generate follow-on work items (`types/recurring-plan.ts`).

## Who uses it

**Teams tiers only** (Small / Mid / Large / Enterprise) for the full drawer set (`docs/features/crm.md`, `lib/drawers.ts`). **Pro** has **clients/properties** but **not** this pipeline per feature tier matrix.

## Data involved

- Collections: `requests`, `quotes`, `jobs`, `invoices` (`types/*.ts`).
- **Team workflow settings:** embedded on **`teams/{teamId}.workflowSettings`** (`getTeamWorkflowSettings` in `lib/firebase/firestore.ts`).
- **`work_items`:** `workflow_*` mirror kinds + optional `workItemIds` on CRM docs (`types/work-item.ts`).
- **Conversion fields:** `convertedToQuoteId`, `convertedToJobId`, etc., on entities.

## UI patterns

Separate list + view + form drawers per stage; **workflow step** colors in sidebar (`workflow-step-colors` rule). **Dashboard** may surface upcoming jobs (`mergeDashboardUpcomingWithWorkItems` when flag on).

## Dependencies

- CRM (client/property required for most docs)
- Trees/zones on line items (`treeIds`, `zoneIds` on line item types)
- Tier: `useTeamsOnlyAccess`
- Weather: `job_reschedule` recommendation type ties **workflow** to **weather.md**

## Inconsistencies & overlaps

- **Job** (workflow document) vs **`WorkItem`** with `workflow_job`—duplicate conceptual layer (`Groundzy v3/05-data/entities.md`).
- **Request** (CRM) vs **tree access request** (`tree_access_requests`)—different entities, same word “request.”
- Historical **audit** noted conversion/backlink gaps; **upgrade** docs added atomic conversions—behavior may have evolved; verify code paths when changing data.
