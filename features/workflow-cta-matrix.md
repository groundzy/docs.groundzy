# Workflow drawer CTAs and stored statuses (matrix)

**Source of truth in code:** [`app/lib/workflow/workflow-action-registry.ts`](../../app/lib/workflow/workflow-action-registry.ts) and [`app/lib/workflow/workflow-action-context.ts`](../../app/lib/workflow/workflow-action-context.ts).  
**UI layout:** [`app/drawers/components/WorkflowDrawerActions.tsx`](../../app/app/drawers/components/WorkflowDrawerActions.tsx) — Row A (toolbar + Mark-as + More + delete), Row B (single primary / `primarySlot`), More → Pipeline / Communications / Billing (when present).

**Derived labels** (not stored on Firestore `status`): request/job schedule stripes (`today` / `upcoming` / `overdue`), invoice **past due** presentation from `dueDate` + `awaiting_payment` — see [`app/lib/workflow/workflow-status.ts`](../../app/lib/workflow/workflow-status.ts).

## Intentional differences vs Jobber (reference: `jobber/jobber-pipeline-reference.md`)

| Area | Jobber | Groundzy |
|------|--------|----------|
| Request pipeline | Assessment-centric statuses (`unscheduled`, `assessment_completed`, …) | CRM ladder (`new`, `contacted`, `quoted`, …) + same derived schedule stripes |
| Job | Separate visit ops vs billing closure | Single `JobStatus` axis + derived visit stripes |
| Quote archive from draft | Must move to awaiting response first | **Same rule:** archive not offered from `draft` in the drawer registry |

---

## Request (`RequestStatus` — `types/request.ts`)

| Stored status | Row B / primary | Toolbar & overflow |
|---------------|-----------------|----------------------|
| Non-terminal (e.g. `new` … `scheduled`) | **Convert to** popover when user can quote/job; else **Continue to quote/job** when already converted | Edit, Delete, Mark-as (non-terminal), More → View quote / View job (when linked), email/text |
| Terminal (`completed`, `converted`, `archived`, `declined`, `cancelled`) | **Continue to quote** or **Continue to job** when backlinks exist and Convert is N/A | Edit read-only / Mark-as off per registry |

---

## Quote (`QuoteStatus`)

| Stored status | Primary | More → Pipeline | More → Communications |
|---------------|---------|-----------------|----------------------|
| `draft` | Convert to job (if approved path N/A — primary is view job OR convert when `approved`) | Archive **hidden** (Jobber: exit draft first) | Send email, send text |
| `awaiting_response`, `changes_requested` | Convert to job when approved; else mark-as | Archive, duplicate quote, copy client link (when allowed) | Send email, text |
| `approved` | Convert to job | Archive, duplicate, client link | Send email, text |
| `converted` | View job | Duplicate (similar), client link | Send email, text |
| `archived`, `declined` | View job if converted | Duplicate where policy allows | … |

Permissions: create-from-quote / quote create / quote update gate respective CTAs (`crm-workflow-ui-gates`, org role matrix).

---

## Job (`JobStatus`)

Unchanged in this pass — see registry `getJobDrawerActions` (Work mode, Mark complete, Create invoice, View invoice, work order comms).

---

## Invoice (`InvoiceStatus`)

**Stored:** `draft` \| `awaiting_payment` \| `paid` \| `bad_debt` — there is **no** stored `past_due`; list/badge “Past due” is derived.

| Stored | Primary | More → Billing (Pipeline section title may read “Billing”) | More → Communications |
|--------|---------|-------------------------------------------|----------------------|
| `draft` | Record payment (if due) | Issue without email | Email, text, copy client portal link |
| `awaiting_payment` | Record payment (if due) | Mark bad debt, Mark paid (no payment recorded)* | Email, text, copy client portal link |
| `paid` | — | — | Copy link (optional) |
| `bad_debt` | — | — | — |

\*Staff-only; requires invoice update permission. Sets `paid` with rollups when “close without payment” is used; `bad_debt` for write-off.

---

## Staging QA (staff vs participant)

1. Open each view drawer as **org staff** — new overflow items appear and succeed (or show permission tooltip).  
2. Confirm **participant** / external surfaces **hide** staff-only sends (existing `isExternalSurface`).  
3. After **bad debt** / **mark paid**, Client Hub / list filters show expected rows (`bad_debt` hidden from client hub per app rules).  
4. **Duplicate quote** opens add-quote with pre-filled copy when `duplicateFromQuoteId` is set.

---

*Last updated: workflow CTA implementation pass.*
