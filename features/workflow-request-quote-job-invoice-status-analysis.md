# Workflow status analysis: Request, Quote, Job, Invoice

**Scope:** Groundzy commercial pipeline (`requests` → `quotes` → `jobs` → `invoices`).  
**Sources:** `app` TypeScript (`types/*.ts`, drawers, `lib/firebase/firestore.ts`, `lib/workflow/*`, Groundzy v3 projection handlers under `lib/groundzy/projections/handlers/workflow/`) and `docs` (`features/current-workflow-audit.md`, `groundzy-v3-docs/05-data/*.md`, `06-features/workflow.md`).  
**Date:** 2026-04-16.

**Ship 2 drawer actions:** Which actions appear in workflow **view** drawers (and their visibility, including Edit vs read-only alignment) is defined in code, not in this document. See **`app/lib/workflow/workflow-action-registry.ts`** (`getRequestDrawerActions`, `getQuoteDrawerActions`, `getJobDrawerActions`, `getInvoiceDrawerActions`), with predicates in `app/lib/workflow/workflow-action-context.ts` and shared layout in `app/drawers/components/WorkflowDrawerActions.tsx`. Details: `docs/features/workflow-status-redesign-spec.md` (appendix: Stored vs derived / Ship 2).

---

## 1. Executive summary

| Entity   | Status enum (canonical) | Enforcement | Primary UI / automation |
|----------|-------------------------|-------------|---------------------------|
| **Request** | `new`, `contacted`, `quoted`, `approved`, `scheduled`, `completed`, `declined`, `cancelled` | No transition matrix; Mark-as excludes current only | Mark-as popover; v3 quote-from-request sets `quoted` + `convertedToQuoteId`; legacy `convertRequestToJob` sets `scheduled` + `convertedToJobId` |
| **Quote** | `draft`, `sent`, `viewed`, `accepted`, `declined`, `expired`, `converted` | Accept/decline gated by status in view | Create → `draft` (projections); Accept/Decline; `convertQuoteToJob` / v3 job-from-quote → `converted` |
| **Job** | `draft`, `scheduled`, `confirmed`, `in_progress`, `on_hold`, `completed`, `cancelled`, `requires_invoicing` | **`lib/workflow/job-status-transitions.ts`** (Mark-as + edit form) | Mark Complete → `completed` + `completedAt`; first invoice may flip `requires_invoicing` → `completed`; Mark Paid on invoice sets job `paidAt` |
| **Invoice** | `draft`, `sent`, `viewed`, `partial_payment`, `paid`, `overdue`, `cancelled` | No transition matrix in app | Create → `draft`; list filters include full enum; **Mark Paid** → `paid` (+ amounts); `updateInvoice` writes job `paidAt` when paid |

**Important implementation facts:**

1. **Two write paths:** Team UI favors **Groundzy v3 append + projections** for creates (`useCreateQuote`, etc.), while **`lib/firebase/firestore.ts`** still contains **deprecated** atomic helpers (`convertRequestToQuote`, `convertQuoteToJob`, `convertJobToInvoice`) used for scripts/migration. Behavior is aligned where both exist (e.g. backlinks and status side effects).

2. **`docs/features/current-workflow-audit.md` is partially stale:** It states conversion backlinks are “never written” and “Mark Complete sets `requires_invoicing`.” Current code writes `convertedToQuoteId` / `convertedToJobId` via v3 projections and legacy batch converts, and **Mark Complete sets `status: "completed"`** (not `requires_invoicing`). Treat the audit as historical context; this report reflects the codebase as of the analysis date.

3. **“Open” helpers** (`lib/workflow-open-filters.ts`): terminal requests = `declined` \| `cancelled` \| `completed`; open quotes = `draft` \| `sent` \| `viewed`; terminal jobs = `completed` \| `cancelled`; terminal invoices = `paid` \| `cancelled`.

---

## 2. Request status

### 2.1 Type definition

Defined in `types/request.ts` as `RequestStatus` (eight values). New requests default to **`new`** in `createRequest` (`lib/firebase/firestore.ts`).

### 2.2 User-driven changes

- **`view-request.tsx`:** “Mark as” lists fixed options (`MARK_AS_OPTIONS`) and **filters out the current status**; any other status can be chosen in one click (`updateRequest` with `{ status }`). There is **no** graph of allowed transitions in code.
- **Request → Quote (v3):** `handleWorkflowQuoteCreatedFromRequest` sets request **`status: "quoted"`** and **`convertedToQuoteId`** (`lib/groundzy/projections/handlers/workflow/quote-from-request.ts`). This matches `useCreateQuote` when `requestId` is passed (`hooks/useQuotes.ts`).
- **Legacy atomic:** `convertRequestToQuote` batch-sets **`convertedToQuoteId`**, **`status: "quoted"`** on the request.
- **Request → Job (legacy atomic):** `convertRequestToJob` sets **`convertedToJobId`** and **`status: "scheduled"`** on the request (not used by the default v3 job-creation path from the UI if that path uses events only—verify when touching job creation).

### 2.3 Documentation alignment

- `groundzy-v3-docs/05-data/entities.md` lists Request with `status` and `convertedToQuoteId?`.
- `groundzy-v3-docs/05-data/data-model-overview.md` does not enumerate request statuses; rely on `types/request.ts`.

---

## 3. Quote status

### 3.1 Type definition

`QuoteStatus` in `types/quote.ts`: `draft`, `sent`, `viewed`, `accepted`, `declined`, `expired`, `converted`. Optional timestamps: `sentAt`, `viewedAt`, `respondedAt` (schema supports customer-facing flows; **not all paths populate them**—grep shows minimal writes outside types).

### 3.2 Creation

- **V3 `quote_created` / quote from request:** Projections set new quotes to **`draft`** (`quote-created.ts`, `quote-from-request.ts`).
- **Legacy `createQuote` in Firestore:** also **`draft`** (if used).

### 3.3 User-driven and conversion updates

- **`view-quote.tsx`:**  
  - **Accept** → `status: "accepted"` then navigate to add-job.  
  - **Decline** → `declined`.  
  - **Create Job (without accepting)** → navigate without forcing accept.  
  - **`canAcceptOrDecline`:** `draft` \| `sent` \| `viewed` (so draft quotes can be accepted/declined without a separate “send” step in UI).
- **`convertQuoteToJob` / v3 `handleWorkflowJobCreatedFromQuote`:** quote becomes **`converted`**, **`convertedToJobId`** set.

### 3.4 List filters and “open” semantics

- **`quotes.tsx`:** Filter dropdown includes `sent`, `viewed`, `expired`, etc.
- **`isOpenQuote`:** true for **`draft` \| `sent` \| `viewed`** only (`lib/workflow-open-filters.ts`). Accepted/declined/expired/converted are treated as non-open for that helper.

### 3.5 External / portal

- **`/api/quote-portal/*`:** Token issuance and public GET for portal; quote lifecycle may evolve independently—see `app/api/quote-portal/`. Not all quote statuses are driven from the main Teams drawers alone.

---

## 4. Job status

### 4.1 Type definition

`JobStatus` in `types/job.ts` with documented semantics (operational states, **`completed`** + `completedAt`, **`requires_invoicing`** as billing queue, **`cancelled`**).

### 4.2 Transition matrix (authoritative)

**File:** `lib/workflow/job-status-transitions.ts`.

- **Display order:** `JOB_STATUS_ORDER` (draft → … → cancelled; includes `requires_invoicing` after `completed` in the ordered list).
- **Mark-as / edit-job allowed one-step moves:** `FROM_STATUS` map. Examples:
  - From **`draft`:** → `scheduled`, `confirmed`, `in_progress`, `cancelled`.
  - From **`completed`:** → `requires_invoicing`, `cancelled` (billing queue is explicit after completion).
  - From **`requires_invoicing`:** → `completed`, `cancelled`.
  - **`cancelled`:** no exits (empty list).
- **`canTransitionJobStatus` / `getSelectableJobStatuses`:** used by **`job-form.tsx`** and **`view-job.tsx`** so users cannot pick arbitrary jumps outside the table.

### 4.3 Mark Complete (explicit action)

**`view-job.tsx` `handleComplete`:** sets **`status: "completed"`** and **`completedAt`** (not `requires_invoicing`). This matches `data-model-overview.md` (“Mark complete … sets `status: completed`”).

**Eligible statuses for the button:** `scheduled` \| `confirmed` \| `in_progress` (`canComplete`).

### 4.4 Invoice side effects on job

- **`convertJobToInvoice`** (`firestore.ts`): batch creates invoice, sets job **`primaryInvoiceId`**, **`invoicedAt`**; if job was **`requires_invoicing`**, patches job **`status` → `completed`**.
- **V3 `handleWorkflowInvoiceCreatedFromJob`:** same idea: if **`jobStatusBeforeInvoice === "requires_invoicing"`**, projection sets job **`status: "completed"`** (in addition to invoice writes).
- **`updateInvoice`** when status becomes **`paid`:** sets linked job **`paidAt`** (`firestore.ts`).

### 4.5 Delete job

`view-job.tsx` references **`deleteJobMutation`**—job deletion exists in UI path (verify rules and `deleteJob` in Firestore); the older audit said “no delete”—may be outdated.

---

## 5. Invoice status

### 5.1 Type definition

`InvoiceStatus` in `types/invoice.ts`: `draft`, `sent`, `viewed`, `partial_payment`, `paid`, `overdue`, `cancelled`.

### 5.2 Creation

- **`createInvoice`** and **`convertJobToInvoice`:** initial **`draft`**, **`amountPaid: 0`**, **`amountDue`** computed.

### 5.3 User-driven updates in main UI

- **`view-invoice.tsx`:** **Mark Paid** sets **`status: "paid"`**, **`amountPaid` = total**, **`amountDue` = 0** (when not already `paid`/`cancelled`).
- **No** edit-invoice drawer in the registry; status changes beyond Mark Paid are not exposed as a full editor in the same way as jobs.

### 5.4 List filters

**`invoices.tsx`:** Select includes `draft`, `sent`, `viewed`, `partial_payment`, `paid`, `overdue`, `cancelled`—the full enum is filterable even if some values are rare without automation (e.g. overdue might need cron or manual data).

### 5.5 Cross-entity: `paid` → job

**`updateInvoice`:** when patched **`status === "paid"`**, calls **`updateJob(jobId, { paidAt: now })`** (`firestore.ts`).

---

## 6. Cross-cutting: dashboard / timeline

- **`mergeWorkflowActiveRows` / `mergeWorkflowTimelineRows`** (`lib/workflow-open-filters.ts`): attach **`workflowStatus`** from each entity for merged lists; quote/job rows from `workflowRowMetaFor*` do not replace status—they add date/assignee metadata (`lib/workflow-active-row-meta.ts`).
- **`WorkflowActiveWorkCard`:** maps raw status strings to badge styling (e.g. terminal vs active patterns).

---

## 7. Docs vs code: reconciliation checklist

| Topic | Docs reference | Code reality (2026-04) |
|-------|------------------|----------------------|
| Conversion backlinks | `current-workflow-audit.md` — “never written” | **Written** by v3 projections (`quote-from-request`, `job-from-quote`) and legacy `convertRequestToQuote` / `convertRequestToJob` / `convertQuoteToJob` |
| Mark Complete job status | Audit appendix — `requires_invoicing` | **`completed`** + `completedAt` in `view-job.tsx` |
| Job lifecycle | `data-model-overview.md` §2 | Aligns with Mark Complete = `completed`, invoice batch moves `requires_invoicing` → `completed`, paid sets `paidAt` |
| Quote “converted” | Audit — “never set” | Set by **`convertQuoteToJob`** and **`handleWorkflowJobCreatedFromQuote`** |

---

## 8. Gaps and recommendations

1. **Request status:** No validation of business ordering (e.g. `new` → `scheduled`); relies on user discipline. Consider documenting intended funnel or adding soft validation.
2. **Quote `sent` / `viewed` / `expired`:** Enum and filters exist; primary drawer flow centers on **`draft`** and accept/decline/convert. If product expects customer-facing send/view tracking, ensure one code path updates these consistently (portal, APIs, or manual).
3. **Invoice `partial_payment`, `overdue`, `sent`, `viewed`:** Same as above—types and filters exist; **Mark Paid** is the main status mutation in `view-invoice.tsx`. Confirm product intent for automation (e.g. overdue by due date).
4. **Documentation maintenance:** Refresh or supersede **`sections of `current-workflow-audit.md`** that contradict v3 projections and current job completion behavior, or add a banner pointing to this analysis.
5. **Naming collision:** “Request” in CRM vs **tree access request** (`tree_access_requests`) — already noted in `entities.md`; keep distinction in new docs.

---

## 9. Key file index (status-related)

| Area | Path |
|------|------|
| Types | `app/types/request.ts`, `quote.ts`, `job.ts`, `invoice.ts` |
| Job transitions | `app/lib/workflow/job-status-transitions.ts` |
| Open / terminal helpers | `app/lib/workflow-open-filters.ts` |
| Firestore CRUD + legacy converts | `app/lib/firebase/firestore.ts` (requests, quotes, jobs, invoices, `convert*` functions) |
| V3 projections | `app/lib/groundzy/projections/handlers/workflow/quote-from-request.ts`, `job-from-quote.ts`, `invoice-from-job.ts`, `quote-created.ts`, `request-created.ts` |
| View drawers | `app/app/drawers/view-request.tsx`, `view-quote.tsx`, `view-job.tsx`, `view-invoice.tsx` |
| Drawer action menus (Ship 2) | `app/lib/workflow/workflow-action-registry.ts`, `workflow-action-context.ts`, `app/drawers/components/WorkflowDrawerActions.tsx` |
| List row menus (Ship 2) | `app/lib/workflow/workflow-list-row-actions.ts` (registry parity + `WorkflowListContextMenu`) |
| Product docs | `docs/groundzy-v3-docs/05-data/data-model-overview.md`, `entities.md`, `docs/groundzy-v3-docs/06-features/workflow.md` |

---

*End of report.*
