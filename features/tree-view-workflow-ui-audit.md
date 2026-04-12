# Tree view workflow UI — audit (chat session changes)

This document audits code touched by the workflow work on the **View tree** drawer **Operations** tab: replacing ad-hoc empty copy with shared workflow UI, performance tweaks, then simplification to a single **Workflow** list.

## Current behavior (after changes)

- **`TreeWorkflowSections`** (`app/drawers/view-tree/components/TreeWorkflowSections.tsx`):
  - If **`workflowEnabled`** is false (no property on tree): shows the **“Operations”** card with copy to link a property and **Edit tree**.
  - If **`workflowEnabled`** is true: renders **`WorkflowActiveWorkCard`** with **`scope="property"`** when `tree.propertyId` is set — same **Workflow** UI as **View property** / **View client**: tabs **All / Requests / Quotes / Jobs / Invoices**, merged timeline data (all statuses, sorted by activity time) with **status badges**, for that property, filtered by client when present.
- **No** per-entity sections (Requests / Quotes / Jobs / Invoices) scoped to “items linked to this tree.”
- **No** inline **Create workflow** menu in this section; creation stays on the drawer footer via **`TreeWorkflowCreateMenu`** in **`ViewTreeActionFooter`**.

## Files changed in this evolution

| Area | Change |
|------|--------|
| `app/drawers/view-tree/components/TreeWorkflowSections.tsx` | Reduced to property **`WorkflowActiveWorkCard`** + property-required card. Removed **`useWorkflowForTree`**, **`TreeWorkflowCreateMenu`**, entity lists, loading row, and **`WorkflowRow`** / **`EntitySection`** helpers. |
| `hooks/useWorkflowForTree.ts` | **Deleted** — was the only consumer of tree-scoped filtering over property lists. |
| `lib/workflow/tree-workflow-constants.ts` | Removed **`WORKFLOW_PROPERTY_LIST_LIMIT`**, **`WORKFLOW_PROPERTY_INVOICE_LIMIT`**, **`WORKFLOW_TREE_MAX_ROWS_PER_SECTION`** (only used by the removed hook / old sections). Kept **`MAX_PROPERTY_JOB_QUERIES`** and **`JOB_MAP_BADGE_ACTIVE_STATUSES`**. |
| `lib/i18n/messages.ts` | **`loadingTreeLinked`** added then removed. Several **`viewTreeWorkflow.*`** strings may now be **unused** (see below). |

## Files intentionally unchanged (related context)

| File | Role |
|------|------|
| `app/drawers/components/WorkflowActiveWorkCard.tsx` | Canonical **Workflow** card (All / per-type tabs, merged R/Q/J/I + status badges); tree reuses it with `scope="property"`. |
| `app/drawers/view-tree/components/TreeOperationsTab.tsx` | Composes **`TreeWorkflowSections`** with `workflowEnabled={!!tree.propertyId}`. |
| `app/drawers/view-tree/components/ViewTreeActionFooter.tsx` | **`TreeWorkflowCreateMenu`** for **Create workflow** (footer). |
| `app/drawers/view-tree/components/TreeWorkflowCreateMenu.tsx` | Popover for add-request / quote / job / invoice with `treeId` + property/client params. |
| `lib/workflow/tree-workflow-links.ts` | Tree reference helpers; still used elsewhere (e.g. map / prefill), **not** tied to the removed hook. |
| `lib/workflow/tree-prefill-labels.ts` | Uses **`viewTreeWorkflow.prefill*`** keys for line-item labels. |

## Removed product surface

- **Tree-linked workflow lists**: Previously, **`useWorkflowForTree`** loaded property-scoped requests/quotes/jobs/invoices and filtered client-side with **`tree-workflow-links`** to show only rows referencing the current tree. That UI is **gone**; users see **property-level** workflow in this panel.
- **Merged list**: All workflow documents for the scope appear in one list (capped per type and overall; see **`mergeWorkflowTimelineRows`** in **`lib/workflow-open-filters.ts`**), with a **status badge** per row.
- **Follow-up (optional)**: A **tree-scoped** workflow strip (only requests/quotes/jobs/invoices that reference the current tree via **`lib/workflow/tree-workflow-links.ts`**) is not implemented; the card remains **property-level** for all scopes.

## Performance notes (historical)

Earlier iterations gated the whole block on **`useWorkflowForTree`** `isLoading`, which delayed showing **Active work**. Later, **Active work** was shown immediately and loading/duplicate-query alignment was discussed; with the hook removed, **`WorkflowActiveWorkCard`** owns its own queries (**`useRequests` / `useQuotes` / `useJobs` / `useInvoices`**, etc.).

## i18n cleanup candidates

These **`viewTreeWorkflow`** keys appear **only** in `lib/i18n/messages.ts` (no `t("…")` usage in TS/TSX found). Safe to remove in a dedicated i18n cleanup if desired:

- `emptyWorkflow`
- `sectionRequests`, `sectionQuotes`, `sectionJobs`, `sectionInvoices`
- `viewAllInRequests`, `viewAllInQuotes`, `viewAllInJobs`, `viewAllInInvoices`

(Keep keys still used by **`TreeWorkflowSections`**, **`TreeWorkflowCreateMenu`**, **`TreeOperationsTab`**, **`tree-prefill-labels`**, etc.)

## Stale documentation

- `.cursor/plans/tree–workflow_integration_*.plan.md` and **`docs/features/current-workflow-audit.md`** may still describe **`useWorkflowForTree`** on **TreeOperationsTab**. Update those when you next edit docs/plans.

## Verification

- Open **View tree** → **Operations** (Teams, tree with property): expect **Active work** card only (plus footer **Create workflow**).
- Tree **without** property: expect assign-property **Operations** card with **Edit tree**.

---

*Generated to reflect the codebase state after the chat-driven changes; adjust if further PRs land.*
