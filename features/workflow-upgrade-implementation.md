# Workflow Upgrade Implementation

> **Document purpose**: Definitive documentation of the Request → Quote → Job → Invoice workflow upgrade implemented in the Groundzy codebase. Describes changes, conversion rules, compatibility, and follow-ups.

---

## 1. Executive Summary

The workflow upgrade transforms Groundzy's four-stage pipeline from **navigation + prefill** semantics into a **true relational pipeline** with:

- **Atomic conversion functions** for Request→Quote, Request→Job, Quote→Job, Job→Invoice
- **Persisted backlinks** (`convertedToQuoteId`, `convertedToJobId`) on source entities
- **Unified line item flow** via mappers that preserve treeIds/zoneIds across stages
- **Edit-invoice support** with editable line items and paid-invoice lock
- **Removal** of misleading Request→Invoice conversion
- **Backward compatibility** with existing production data

The architecture remains drawer-based. No destructive migrations are required.

---

## 2. Summary of Changes

| Area | Change |
|------|--------|
| **Conversion logic** | Firestore `convertRequestToQuote`, `convertRequestToJob`, `convertQuoteToJob`, `convertJobToInvoice` with batched writes |
| **Backlinks** | Request.convertedToQuoteId, Request.convertedToJobId, Quote.convertedToJobId written on conversion |
| **Line items** | `lib/workflow/line-item-mappers.ts` – RequestItem→QuoteLineItem→JobLineItem→InvoiceLineItem |
| **Hooks** | useCreateQuote, useCreateJob, useCreateInvoice call conversion functions when source IDs present |
| **Forms** | quote-form, job-form submit `{ data, requestId }` / `{ data, quoteId, requestId }`; prefill via mappers |
| **View drawers** | view-request/view-quote/view-job show View Quote/View Job when converted; Request→Invoice removed |
| **Edit invoice** | edit-invoice drawer, invoice-form edit mode, editable line items, paid-invoice lock |
| **Types** | QuoteLineItem, JobLineItem, InvoiceLineItem optional `zoneIds?: string[]` |
| **i18n** | invoiceForm.update, saving, toastInvoiceUpdated, toastUpdateFailed; edit-invoice drawer label (en/es/fr) |

---

## 3. Conversion Rules Implemented

### 3.1 Request → Quote

- **Precondition**: Request exists, `request.convertedToQuoteId` is absent
- **Actions**:
  1. Create Quote with `requestId`
  2. Update Request: `convertedToQuoteId`, `status: "quoted"`
- **Line items**: If form sends none, derive from `request.requestItems` via `requestItemsToQuoteLineItems()`
- **Backlink**: `Request.convertedToQuoteId`

### 3.2 Request → Job

- **Precondition**: Request exists, `request.convertedToJobId` is absent
- **Actions**:
  1. Create Job with `requestId`
  2. Update Request: `convertedToJobId`, `status: "scheduled"`
- **Line items**: If job pricing empty, derive Request→Quote→Job line items via mappers
- **Backlink**: `Request.convertedToJobId`

### 3.3 Quote → Job

- **Precondition**: Quote exists, `quote.convertedToJobId` is absent
- **Actions**:
  1. Create Job with `quoteId`, `requestId` (from quote)
  2. Update Quote: `convertedToJobId`, `status: "converted"`
- **Line items**: If job pricing empty, copy `quote.lineItems` via `quoteLineItemsToJobLineItems()`
- **Backlink**: `Quote.convertedToJobId`

### 3.4 Job → Invoice

- **Precondition**: Job exists
- **Actions**: Create Invoice with `jobId`, `quoteId` (from job when available)
- **Line items**: If form sends none, copy `job.pricing.lineItems` via `jobLineItemsToInvoiceLineItems()`
- **Backlink**: None ( Invoice→Job is the primary relationship; invoice holds `jobId`)

### 3.5 Removed Conversion

- **Request → Invoice**: Removed. Invoice requires a Job; direct conversion was misleading.

---

## 4. Field Mapping Per Transition

### Request → Quote

| Source | Destination |
|--------|-------------|
| organizationId, clientId, propertyId | Quote |
| requestItems | → lineItems (via requestItemsToQuoteLineItems) |
| request.id | quote.requestId |

### Quote → Job

| Source | Destination |
|--------|-------------|
| organizationId, clientId, propertyId | Job |
| quote.lineItems | → pricing.lineItems (via quoteLineItemsToJobLineItems) |
| quote.requestId | job.requestId |
| quote.id | job.quoteId |
| treeIds from line items | job.treeIds |

### Job → Invoice

| Source | Destination |
|--------|-------------|
| organizationId, clientId | Invoice |
| job.pricing.lineItems | → lineItems (via jobLineItemsToInvoiceLineItems) |
| job.id | invoice.jobId |
| job.quoteId | invoice.quoteId |

---

## 5. New / Updated Types

### Line Item Additions

- **QuoteLineItem**, **JobLineItem**, **InvoiceLineItem**: Optional `zoneIds?: string[]` for zone continuity

### Types Unchanged (backlinks already existed)

- **Request**: `convertedToQuoteId?`, `convertedToJobId?`
- **Quote**: `convertedToJobId?`

---

## 6. Compatibility Notes

- **Existing records** without backlinks: UI treats `undefined` as unconverted; shows Convert actions
- **Existing records** with backlinks: UI shows View Quote / View Job
- **Existing quote/job line items**: Mappers tolerate missing treeIds/zoneIds
- **Existing invoices**: Read-only behavior preserved; edit only for non-paid invoices

### No Migration Required

- All new fields are optional
- Missing `convertedTo*Id` defaults to “no conversion” in logic
- No destructive schema changes

### Migration Script (Optional)

If you want to backfill backlinks for existing converted records (e.g. where quote.requestId exists but request.convertedToQuoteId does not), a separate migration script can be written. Runtime code does **not** depend on such migration.

---

## 7. Key Files Changed

| File | Changes |
|------|---------|
| `lib/workflow/line-item-mappers.ts` | **New** – requestItemsToQuoteLineItems, quoteLineItemsToJobLineItems, jobLineItemsToInvoiceLineItems, extractTreeIdsFromLineItems |
| `lib/firebase/firestore.ts` | **Added** – convertRequestToQuote, convertRequestToJob, convertQuoteToJob, convertJobToInvoice |
| `hooks/useQuotes.ts` | useCreateQuote uses convertRequestToQuote when requestId provided |
| `hooks/useJobs.ts` | useCreateJob uses convertQuoteToJob/convertRequestToJob when quoteId/requestId provided |
| `hooks/useInvoices.ts` | useCreateInvoice always uses convertJobToInvoice |
| `app/drawers/quote-form.tsx` | Submits requestId; prefill from request via mappers |
| `app/drawers/job-form.tsx` | Submits quoteId/requestId; prefill from quote via mappers |
| `app/drawers/invoice-form.tsx` | Edit mode, editable line items, paid-invoice lock |
| `app/drawers/view-request.tsx` | View Quote/Job, remove Request→Invoice, conditional convert |
| `app/drawers/view-quote.tsx` | View Job when converted |
| `app/drawers/view-job.tsx` | View Invoice when invoice exists for job |
| `app/drawers/view-invoice.tsx` | Edit button (disabled when paid) |
| `lib/drawers.ts` | Registered `edit-invoice` drawer |
| `types/quote.ts`, `types/job.ts`, `types/invoice.ts` | zoneIds on line items |
| `lib/i18n/messages.ts` | invoiceForm.update, saving, toastInvoiceUpdated, toastUpdateFailed; edit-invoice label |

---

## 8. Drawer Registration

- **edit-invoice**: `requiredProps: ["invoiceId"]`, same tier visibility as add-invoice

---

## 9. Paid Invoice Restriction

- **View invoice**: Edit button disabled when `status === "paid"`
- **Edit form**: Guard blocks edit when paid; shows explanatory message
- **Business rule**: Paid invoices are non-editable; documented in code

---

## 10. Open Follow-ups

1. **Zone linkage**: zoneIds carried through mappers; zone UI in Quote/Job/Invoice forms can be enhanced later
2. **Job status on invoice**: Job does not yet update status when invoice created; can add if desired
3. **Optional backfill migration**: Script to populate backlinks for pre-upgrade conversions
4. **Query invalidation**: Ensure React Query / SWR cache invalidation after conversions; verify in testing

---

## 11. Verification Flows

| Flow | Expected |
|------|----------|
| A: Request → Quote | quote.requestId set, request.convertedToQuoteId set, View Quote shown |
| B: Request → Job | job.requestId set, request.convertedToJobId set |
| C: Quote → Job | job.quoteId, job.requestId, quote.convertedToJobId, quote status "converted" |
| D: Job → Invoice | invoice.jobId, invoice.quoteId (if job has it), line items copied |
| E: Edit Invoice | Edits persist; paid invoices blocked |
| F: Legacy data | Quotes/jobs without backlinks still load; UI shows Convert |

---

## 12. Implementation Report

### Files Changed

- **Created**: `lib/workflow/line-item-mappers.ts`, `docs/features/workflow-upgrade-implementation.md`
- **Modified**: `lib/firebase/firestore.ts`, `hooks/useQuotes.ts`, `hooks/useJobs.ts`, `hooks/useInvoices.ts`, `app/drawers/quote-form.tsx`, `app/drawers/job-form.tsx`, `app/drawers/invoice-form.tsx`, `app/drawers/view-request.tsx`, `app/drawers/view-quote.tsx`, `app/drawers/view-job.tsx`, `app/drawers/view-invoice.tsx`, `lib/drawers.ts`, `types/quote.ts`, `types/job.ts`, `types/invoice.ts`, `lib/i18n/messages.ts`

### Key Decisions

- **Batched writes**: Conversion uses Firestore `writeBatch` for atomicity
- **Duplicate prevention**: Backlink checks in conversion functions; UI hides Convert when backlink exists
- **Request→Invoice**: Removed; no guided flow added (Option A from spec)
- **Line items**: Shared mappers; forms can send custom line items; mappers used only when form sends none/minimal
- **Paid invoices**: Non-editable; Edit disabled in view and guarded in form

### Partially Completed

- None. All specified objectives implemented.

### Risks Left

- **Concurrent conversion**: Two users converting the same Request to Quote simultaneously; Firestore write will succeed for first, second will throw. Acceptable for typical usage.
- **Cache invalidation**: Manual testing recommended for conversion flows; ensure hooks invalidate correctly.
