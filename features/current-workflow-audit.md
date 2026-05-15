# Groundzy Current Workflow Audit

> **Document purpose**: Definitive documentation of the Request → Quote → Job → Invoice workflow as currently implemented in the codebase. Evidence-based, no speculation.

> **Update (April 2026):** CRM **status** strings and **stored vs derived** rules are defined in the app (`types/*.ts`, `lib/workflow/workflow-status.ts`) and in [`workflow-status-redesign-spec.md`](./workflow-status-redesign-spec.md) (appendix). Treat the **status model** table below as **historical context**; confirm against source when auditing behavior.

---

## 1. Executive Summary

The Groundzy workflow system provides a four-stage pipeline for tree care service businesses: **Request** (lead/customer inquiry), **Quote** (estimation and pricing), **Job** (scheduled work), and **Invoice** (billing and payment). All stages are implemented with list, view, and add/edit drawers.

- **Implemented stages**: Request, Quote, Job, Invoice (all four present).
- **Transitions**: Navigation-based prefilling only. No atomic conversion functions. Source entity IDs are passed via URL params and stored as foreign keys (requestId, quoteId, jobId). Back-linking fields (`convertedToQuoteId`, `convertedToJobId`, `convertedToJobId` on Quote) exist in types but are **never written** by the app.
- **Maturity**: Partially mature. CRUD works; conversion is semi-manual with no persistence of conversion relationships on the source entity. Invoice has no edit flow.

---

## 2. Workflow Overview

| Stage   | Entity  | Collection | Status model (stored on `status`)    |
|---------|---------|------------|--------------------------------------|
| Request | Request | `requests` | See `RequestStatus` in `types/request.ts` (includes e.g. `converted`, `archived`). |
| Quote   | Quote   | `quotes`   | `draft`, `awaiting_response`, `approved`, `changes_requested`, `converted`, `archived`, `declined` |
| Job     | Job     | `jobs`     | `unscheduled`, `scheduled`, `action_required`, `requires_invoicing`, `archived` |
| Invoice | Invoice | `invoices` | `draft`, `awaiting_payment`, `paid`, `bad_debt` |

**Relationship model**:
- Quote: `requestId?` → Request
- Job: `requestId?`, `quoteId?` → Request, Quote
- Invoice: `jobId` (required), `quoteId?` → Job, Quote

**Current usage pattern inferred from code**:
1. User creates a Request (lead) or enters workflow at Quote/Job.
2. From Request detail, user clicks "Convert to" → Quote or Job; navigates to add-quote/add-job with `requestId`, `clientId`, `propertyId` in URL.
3. Quote form pre-fills client/property from params; stores `requestId` in Firestore.
4. From Quote detail, user "Accept & Create Job" or "Create Job (without accepting)"; navigates to add-job with `quoteId`, `requestId`, `clientId`, `propertyId`.
5. Job form stores `quoteId`, `requestId` when provided.
6. From Job detail (completed/requires_invoicing), user "Create Invoice"; navigates to add-invoice with `jobId`.
7. Invoice form requires a Job; pre-fills from job pricing; creates invoice with `jobId`, `quoteId`.

**Conversion semantics**: There is **no** formal conversion that atomically creates a child and updates the parent. Conversion = navigation + prefilling + manual form submission. Back-links on source entities are not maintained.

---

## 3. Drawer Architecture

### Registry source
- **lib/drawers.ts** – all workflow drawers registered
- **lib/drawer-registry.ts** – metadata, tier filtering, lazy loading
- **lib/drawer-utils.ts** – `buildDrawerUrl`, `parseDrawerParams`, `drawerIdToUrlParam`

### Workflow drawer matrix

| Drawer ID      | File                 | Archetype | Required params | Tier visibility                     | Sidebar | Bottom nav |
|----------------|----------------------|-----------|-----------------|-------------------------------------|---------|------------|
| requests       | app/drawers/requests.tsx | list   | none            | Small Team, Mid Team, Large Team, Enterprise | yes | no         |
| view-request   | app/drawers/view-request.tsx | detail | requestId       | same                                | no (hideFromNav) | no  |
| add-request    | app/drawers/request-form.tsx | form  | none            | same                                | no      | no         |
| edit-request   | app/drawers/request-form.tsx | form  | requestId       | same                                | no      | no         |
| quotes         | app/drawers/quotes.tsx | list   | none            | same                                | yes     | no         |
| view-quote     | app/drawers/view-quote.tsx | detail | quoteId       | same                                | no      | no         |
| add-quote      | app/drawers/quote-form.tsx | form  | none            | same                                | no      | no         |
| edit-quote     | app/drawers/quote-form.tsx | form  | quoteId       | same                                | no      | no         |
| jobs           | app/drawers/jobs.tsx  | list      | none            | same                                | yes     | no         |
| view-job       | app/drawers/view-job.tsx | detail  | jobId        | same                                | no      | no         |
| add-job        | app/drawers/job-form.tsx | form   | none            | same                                | no      | no         |
| edit-job       | app/drawers/job-form.tsx | form   | jobId        | same                                | no      | no         |
| invoices       | app/drawers/invoices.tsx | list  | none            | same                                | yes     | no         |
| view-invoice   | app/drawers/view-invoice.tsx | detail | invoiceId   | same                                | no      | no         |
| add-invoice    | app/drawers/invoice-form.tsx | form | none            | same                                | no      | no         |

**Note**: There is **no** `edit-invoice` drawer. Invoices cannot be edited after creation.

### Org CRM profile (`crmWorkflowProfile`) — QA checklist (stages + core modules)

Team owners/admins configure **pipeline stages** and **core modules** (clients / properties) under **Settings → Organization → CRM workflow**. The app should not show dead-end CTAs when a stage or module is off. Use this matrix for manual QA (Teams org; adjust toggles in team settings between runs).

| Area | Clients **off** | Properties **off** | Stage **off** (e.g. quotes) |
|------|-----------------|--------------------|-----------------------------|
| **Team settings save** | Turning **requests** on while **clients** is off should be **blocked** (validation). Same for **quotes/jobs** if **clients** or **properties** would be required. | Turning **quotes** or **jobs** on while **properties** is off should be **blocked**. | (Stages can be toggled per product rules.) |
| **Request / quote / job forms** | Client picker hidden; section title reflects **property-only** or **hint** when neither surface applies; **Save ▾** email path hidden without client + email. | Property picker hidden; title **client-only** or hint. | N/A |
| **Invoice form** | **Save ▾** gated on resolvable client email (job/client context). | — | — |
| **Property form** (`add-property` / `edit-property`) | **Client *** picker hidden; create/update does not require `clientId` (`propertyFormShowsOrgClientPicker`). Edit shows read-only **Linked client** when the property still has a `clientId` from before the module was turned off. | Redirect / toast when properties module off (unchanged). | N/A |
| **View request** | Convert to quote/job hidden when target stage or create action not allowed for the org profile + role. | — | Convert actions that target disabled stages hidden. |
| **View quote** | — | — | Accept / create job gated when **job** stage or action disallowed. |
| **View job** | Client/property deep links respect core module flags (labels still show from snapshot). | Same. | Create invoice hidden when **invoice** stage or action disallowed; quote/request buttons unchanged if IDs exist. |
| **Nav / tree / org create menus** | Rows for add-request / add-quote / add-job / add-invoice filtered with the same rules as quick-add (`isWorkflowAddDrawerAllowed` + org create matrix). Invoice row requires **job** stage on and **client + property** in URL context when the menu supplies params. | Same. | Menu row for that drawer omitted. |
| **Dashboard Upcoming (participant)** | — | — | Request/quote/job participant rows for disabled stages omitted from **today** merge. |
| **Server / policy** | **Append workflow event** / creates that assume disabled modules or stages should be **rejected** (policy layer uses normalized CRM profile + core-module checks). | Same. | Same. |

**Deep links**: Bookmarked `add-quote` (or similar) with the stage off should match existing access-denied / read-only patterns; server must still reject illegal creates.

### Drawer chain pattern
- **List** → click item → `navigate("view-*", { *Id })`
- **List** → Add button → `navigate("add-*")`
- **View** → Edit button → `navigate("edit-*", { *Id })`
- **Form** (add/edit) → Cancel → `navigate("view-*")` or list
- **Form** → Submit → `navigate("view-*", { *Id })` for new entity

### Mobile behavior
- `defaultMobileState`: `"full"` for list/detail; `"half"` for view-request, view-quote, view-job, view-invoice
- `extendOverBottomNav: true` for add-request, add-quote, add-job, add-invoice
- `hideFromNav: true` for all view/add/edit drawers (opened from list or conversion actions only)

---

## 4. Request System

### Data model
- **types/request.ts**
- Fields: id, organizationId, clientId, propertyId?, title, description?, source, sourceDetails?, requestType, priority, serviceDetails?, requestItems?, status, requestedDate?, flexibility?, scheduleStartDate/EndDate/StartTime/EndTime, scheduleAnyTime?, assignedToUserIds?, convertedToQuoteId?, convertedToJobId?, createdAt, updatedAt, createdBy

### Form fields (request-form.tsx)
- Client (required), Property (optional)
- Title, Service Details (multi-select: pruning, removal, stump_grinding, planting, treatment, assessment, other)
- Trees & Zones: per-property trees and zones with per-item service types (collapsible)
- Description
- Schedule: start/end date, start/end time, "Any time" checkbox
- Assignment: multi-select team assignees
- Intake: source, type, priority, source details

### Validation
- Client required
- Title required
- No server-side validation beyond Firestore rules

### UI structure
- **List (requests.tsx)**: Search, status filter, client filter, Add button. Cards show title, status, client, address, date, time, assignees.
- **Detail (view-request.tsx)**: Header with title and status; cards for description, client, property, service details, trees/zones (with map links), schedule, assignment, details. Footer: Edit, Mark as (status popover), Delete, Convert to (Quote / Job / Invoice).
- **Form (request-form.tsx)**: Card-based layout; Client & property, Request, Schedule, Assignment, Intake & priority.

### Mutations/hooks
- `useRequests(organizationId, filters)` – list
- `useRequest(requestId)` – single
- `useCreateRequest()`, `useUpdateRequest()`, `useDeleteRequest()`

### Status model
- new, contacted, quoted, approved, scheduled, completed, declined, cancelled
- Mark-as popover allows changing to any status except current

### Relationships
- **clientId** (required): SearchableSelect, links to view-client
- **propertyId** (optional): SearchableSelect filtered by clientId, links to view-property
- **requestItems**: `{ treeId? | zoneId?, serviceTypes[] }[]` – links to trees and zones with per-item services
- Map integration: "Show on map" for trees/zones; `useMapWorkflowTreePick` for map-click to add tree to request

### Conversion entry points to Quote
- "Convert to" → Quote: `navigate("add-quote", { requestId, clientId, propertyId })`
- **Note**: `convertedToQuoteId` is never written. Quote stores `requestId` when created from this flow.

---

## 5. Quote System

### Data model
- **types/quote.ts**
- Fields: id, organizationId, quoteNumber, clientId, propertyId (required), requestId?, title, description?, lineItems, subtotal, taxRate, taxAmount, discountAmount?, totalAmount, currency, validUntil, paymentTerms?, status, convertedToJobId?, sentAt?, viewedAt?, respondedAt?, createdAt, updatedAt, createdBy

### Form fields (quote-form.tsx)
- Client (required), Property (required)
- Title, Valid until (DatePicker with presets)
- Line items via `WorkflowLineItemsEditor` (mode="quote")
- Tax rate toggle and input
- Payment terms, Notes

### Validation
- Client, property, title required
- Line items filtered to non-empty before submit

### UI structure
- **List (quotes.tsx)**: Search, status filter, client filter, Add. Cards show title, status, total, client.
- **Detail (view-quote.tsx)**: Client link, line items, total, notes. Actions: Accept & Create Job, Decline, Create Job (without accepting). Footer: Edit.
- **Form (quote-form.tsx)**: Client & property, Quote details, Line items, Pricing (tax), Terms & notes.

### Mutations/hooks
- `useQuotes(organizationId, filters)`
- `useQuote(quoteId)`
- `useCreateQuote()`, `useUpdateQuote()`
- **No** delete quote (not implemented)

### Status model
- draft, sent, viewed, accepted, declined, expired, converted
- Accept sets status to "accepted" then navigates to add-job

### Relationships
- **clientId**, **propertyId**: required, SearchableSelect
- **requestId**: optional, passed from params when converting from request
- **lineItems**: `QuoteLineItem[]` with id, name?, description, quantity, unitPrice, taxable?, total, serviceType?, treeIds?

### Conversion entry points to Job
- "Accept & Create Job": updates quote status to accepted, then `navigate("add-job", { quoteId, requestId?, clientId, propertyId })`
- "Create Job (without accepting)": same navigation without status change
- **Note**: `convertedToJobId` is checked in UI (`!quote.convertedToJobId`) but **never written** when a job is created from a quote.

---

## 6. Job System

### Data model
- **types/job.ts**
- Fields: id, organizationId, jobNumber, clientId, propertyId, requestId?, quoteId?, title, description?, category?, status, schedule (startDate, endDate?, estimatedDuration?), pricing?, treeIds?, completedAt?, createdAt, updatedAt, createdBy

### Form fields (job-form.tsx)
- Client (required), Property (required)
- Title, Description
- Schedule: start date, end date, estimated duration (hours)
- Line items via `WorkflowLineItemsEditor` (mode="job") with type dropdown (labor, material, equipment, disposal, other)

### Validation
- Client, property, title, start date required
- Line items optional; total derived from items

### UI structure
- **List (jobs.tsx)**: Search, status filter, client filter, Add. Cards show title, status, date, client.
- **Detail (view-job.tsx)**: Description, client link, schedule, pricing. Actions: Mark Complete (→ requires_invoicing), Create Invoice. Footer: Edit.
- **Form (job-form.tsx)**: Client & property, Job details, Schedule, Pricing (line items).

### Mutations/hooks
- `useJobs(organizationId, filters)`
- `useJob(jobId)`
- `useCreateJob()`, `useUpdateJob()`
- **No** delete job (not implemented)

### Status model
- draft, scheduled, confirmed, in_progress, on_hold, completed, cancelled, requires_invoicing
- Mark Complete sets status to `requires_invoicing` and `completedAt`

### Relationships
- **clientId**, **propertyId**: required
- **requestId?**, **quoteId?**: optional, from conversion params
- **pricing.lineItems**: `JobLineItem[]` with treeIds? per item
- **treeIds**: aggregated from line item treeIds

### Conversion entry points to Invoice
- "Create Invoice": `navigate("add-invoice", { jobId, quoteId?, clientId })`
- **Note**: No back-link from Invoice to Job; Invoice stores `jobId` (required).

---

## 7. Invoice System

### Data model
- **types/invoice.ts**
- Fields: id, organizationId, invoiceNumber, clientId, jobId (required), quoteId?, title, description?, lineItems, subtotal, taxRate, taxAmount, discountAmount?, totalAmount, currency, dueDate, paymentTerms?, status, amountPaid, amountDue, payments?, sentAt?, viewedAt?, createdAt, updatedAt, createdBy

### Form fields (invoice-form.tsx)
- **Job selection**: Required when not coming from view-job (SearchableSelect of invoicable jobs: completed, requires_invoicing)
- Title (pre-filled from job)
- Line items: **read-only** – pre-filled from job pricing
- Tax rate toggle and input
- Due date, Payment terms, Notes

### Validation
- organizationId, job (or activeJob), title required
- Line items not editable; copied from job

### UI structure
- **List (invoices.tsx)**: Search, status filter, client filter, Add. Cards show title, status, total, client.
- **Detail (view-invoice.tsx)**: Client link, due date, line items, subtotal, tax, total, paid, amount due, notes. Action: Mark Paid. **No Edit button.**
- **Form (invoice-form.tsx)**: Job selector (if not from job), Invoice details, Line items (read-only), Pricing, Terms & notes.

### Mutations/hooks
- `useInvoices(organizationId, filters)`
- `useInvoiceDetail(invoiceId)` (note: hook naming inconsistent with others: `useJob`, `useQuote`, `useRequest`)
- `useCreateInvoice()`, `useUpdateInvoice()`
- **No** delete invoice
- **No** edit-invoice drawer; invoices cannot be edited after creation

### Status model
- draft, sent, viewed, partial_payment, paid, overdue, cancelled
- Mark Paid sets status to paid, amountPaid = totalAmount, amountDue = 0

### Relationships
- **jobId** (required): Invoice must be tied to a job
- **quoteId?**: optional, passed from job when creating
- **clientId**: derived from job
- **lineItems**: pre-filled from job.pricing.lineItems; cannot be edited in form

### Gaps
- **Partially implemented**: Invoice form `WorkflowLineItemsEditor` is `readOnly` and `onLineItemsChange={() => {}}` – line items cannot be edited
- **Missing**: edit-invoice drawer and Edit action in view-invoice

---

## 8. Forms and Shared Components

### Shared form architecture
- Card-based layout: `Card`, `CardHeader`, `CardTitle`, `CardContent`
- Section icons in rounded `bg-[hsl(var(--tea-green)/0.15)]` boxes
- `DrawerFooter` with Cancel + Submit
- Scrollable body: `flex-1 min-h-0 overflow-y-auto scroll-styled p-4`

### Shared field components
- **SearchableSelect**: client, property, job selection; supports `addOption` to navigate to add-client/add-property
- **DatePicker**: with `quickPresets` where applicable
- **Input**, **Textarea**, **Label**
- **Switch**: tax toggle
- **Select** / **SelectContent** / **SelectItem**: status filters, source, type, priority

### WorkflowLineItemsEditor
- **File**: `components/workflow/WorkflowLineItemsEditor.tsx`
- **Modes**: request | quote | job | invoice
- **Props**: propertyId, organizationId, treesInProperty, lineItems, onLineItemsChange, currency, locale, showTypeDropdown?, readOnly?, lineItemTemplates?
- **Features**: Add/remove rows; template insertion; tree linking per line item (quote, job); type dropdown (job)
- **Invoice mode**: used with `readOnly` and no-op `onLineItemsChange` – displays job line items only

### Header/footer patterns
- **DrawerBody**, **DrawerFooter** from `components/drawer-layout`
- Sticky footer: `className="bg-background/95 backdrop-blur-sm border-border"`
- View drawers: footer with Edit button; list drawers: header with search, filters, Add

### Loading/error/empty states
- Loading: spinner (`animate-spin rounded-full h-8 w-8 border-b-2 border-primary`)
- Empty: centered text ("No requests yet", "No quotes match your search", etc.)
- Error: toast via sonner

---

## 9. Hooks and Data Fetching

### useRequests
- **File**: hooks/useRequests.ts
- **Queries**: `["requests", organizationId, filters]`, `["request", requestId]`
- **Mutations**: create, update, delete
- **Invalidation**: ["requests"], ["request", requestId] on mutations
- **Firestore**: listRequests, getRequest, createRequest, updateRequest, deleteRequest
- **Filters**: status?, clientId?, limit?

### useQuotes
- **File**: hooks/useQuotes.ts
- **Queries**: `["quotes", organizationId, filters]`, `["quote", quoteId]`
- **Mutations**: create, update (no delete)
- **Invalidation**: ["quotes"], ["quote", quoteId], ["requests"] on create/update
- **Filters**: status?, clientId?, requestId?, limit?

### useJobs
- **File**: hooks/useJobs.ts
- **Queries**: `["jobs", organizationId, filters]`, `["job", jobId]`
- **Mutations**: create, update (no delete)
- **Invalidation**: ["jobs"], ["job", jobId], ["quotes"], ["requests"], ["invoices"] on update
- **Filters**: status?, clientId?, propertyId?, limit?

### useInvoices
- **File**: hooks/useInvoices.ts
- **Queries**: `["invoices", organizationId, filters]`, `["invoice", invoiceId]` (hook: `useInvoiceDetail`)
- **Mutations**: create, update (no delete)
- **Invalidation**: ["invoices"], ["invoice", invoiceId], ["jobs"] on create
- **Filters**: status?, clientId?, jobId?, limit?

### Supporting hooks
- **useClients**: list clients for selectors
- **useProperties**: list properties filtered by clientId
- **useClient**: single client
- **useTrees**: for request/job/quote tree linking
- **useZones**: for request zone linking
- **useTeamAssignees**: for request assignment
- **useWorkflowSettings**: default tax, currency, payment terms presets, line item templates
- **useUserPreferences**: timezone for date/time display
- **useTeamsOnlyAccess**: `hasAccess`, `showUpgradePrompt` for workflow gating

### Important assumptions
- organizationId from `organization.id`, `userDoc.organizationId`, or `user.uid` (fallback for solo users)
- No optimistic updates; mutations invalidate queries
- staleTime: 2 minutes for all workflow queries

---

## 10. Type System

### Request
- `Request`, `CreateRequestData`, `UpdateRequestData`
- `RequestStatus`, `RequestSource`, `RequestType`, `RequestServiceType`
- `RequestItem`: `{ treeId? | zoneId?, serviceTypes }`
- `getRequestAssigneeUserIds()` – normalizes legacy `assignedToUserId`

### Quote
- `Quote`, `CreateQuoteData`, `UpdateQuoteData`
- `QuoteStatus`
- `QuoteLineItem`: id, name?, description, quantity, unitPrice, taxable?, total, serviceType?, treeIds?

### Job
- `Job`, `CreateJobData`, `UpdateJobData`
- `JobStatus`, `JobCategory`
- `JobLineItem`: id, name?, description, quantity, unitPrice, total, type?, treeIds?

### Invoice
- `Invoice`, `CreateInvoiceData`, `UpdateInvoiceData`
- `InvoiceStatus`
- `InvoiceLineItem`, `InvoicePayment`

### Mismatches
- `convertedToQuoteId`, `convertedToJobId` on Request: declared, **never written**
- `convertedToJobId` on Quote: declared, **never written**; UI checks it
- Invoice `lineItems` in form: pre-filled from job, read-only; type supports full editing

---

## 11. Data Model / Firestore Mapping

### requests
- **Collection**: `requests`
- **Fields**: organizationId, clientId, propertyId?, title, description?, source, requestType, priority, serviceDetails?, requestItems?, status, schedule*, assignedToUserIds?, createdAt, updatedAt, createdBy
- **Indexes**: organizationId+status, organizationId+clientId, organizationId+createdAt(desc)
- **Soft delete**: None (hard delete)
- **Scoping**: organizationId

### quotes
- **Collection**: `quotes`
- **Fields**: organizationId, quoteNumber, clientId, propertyId, requestId?, title, description?, lineItems, subtotal, taxRate, taxAmount, totalAmount, currency, validUntil, paymentTerms?, status, createdAt, updatedAt, createdBy
- **Indexes**: organizationId+status, organizationId+clientId, organizationId+requestId, organizationId+createdAt(desc)
- **Scoping**: organizationId

### jobs
- **Collection**: `jobs`
- **Fields**: organizationId, jobNumber, clientId, propertyId, requestId?, quoteId?, title, description?, category?, status, schedule, pricing?, treeIds?, completedAt?, createdAt, updatedAt, createdBy
- **Indexes**: organizationId+status, organizationId+clientId, organizationId+createdAt(desc)
- **propertyId filter** used but composite index for organizationId+propertyId not explicitly in indexes.json – **potential gap**
- **Scoping**: organizationId

### invoices
- **Collection**: `invoices`
- **Fields**: organizationId, invoiceNumber, clientId, jobId, quoteId?, title, description?, lineItems, subtotal, taxRate, taxAmount, totalAmount, currency, dueDate, paymentTerms?, status, amountPaid, amountDue, createdAt, updatedAt, createdBy
- **Indexes**: organizationId+status, organizationId+clientId, organizationId+jobId, organizationId+createdAt(desc)
- **Scoping**: organizationId

---

## 12. Conversion / Transition Logic

### Request → Quote

| Aspect | Current implementation |
|--------|------------------------|
| UI action | View Request → "Convert to" → Quote |
| Location | view-request.tsx, Popover "Convert to" |
| Handler | `handleConvertToQuote()` → `navigate("add-quote", { requestId, clientId, propertyId })` |
| Data flow | add-quote reads `params.requestId`; createQuote stores `requestId` in payload |
| Source back-link | **None**. `Request.convertedToQuoteId` never written |
| Fields copied | None automatically; form pre-fills client/property from params; user must enter line items |
| Process | Semi-manual: navigate → fill form → submit |

### Request → Job

| Aspect | Current implementation |
|--------|------------------------|
| UI action | View Request → "Convert to" → Job |
| Handler | `handleConvertToJob()` → `navigate("add-job", { requestId, clientId, propertyId })` |
| Data flow | add-job reads params; createJob stores `requestId` |
| Source back-link | **None**. `Request.convertedToJobId` never written |
| Fields copied | clientId, propertyId passed; no automatic copy of requestItems to job line items |
| Process | Semi-manual |

### Request → Invoice

| Aspect | Current implementation |
|--------|------------------------|
| UI action | View Request → "Convert to" → Invoice |
| Handler | `handleConvertToInvoice()` → `navigate("add-invoice", { clientId })` – **no requestId, no propertyId** |
| Data flow | Invoice **requires jobId**; add-invoice shows job selector. Request cannot directly create invoice |
| Gap | Invoice cannot be created from Request alone; user must select a job. Request→Invoice is misleading in UI |

### Quote → Job

| Aspect | Current implementation |
|--------|------------------------|
| UI action | View Quote → "Accept & Create Job" or "Create Job (without accepting)" |
| Handler | Accept: updateQuote(status: "accepted") → navigate("add-job", { quoteId, requestId?, clientId, propertyId }). Convert: same navigate |
| Data flow | createJob stores `quoteId`, `requestId` when provided |
| Source back-link | **None**. `Quote.convertedToJobId` never written. View checks `!quote.convertedToJobId` to show actions – always true in practice |
| Fields copied | clientId, propertyId from quote; line items **not** copied from quote to job – user re-enters in job form |
| Process | Semi-manual; no line item transfer |

### Job → Invoice

| Aspect | Current implementation |
|--------|------------------------|
| UI action | View Job → "Create Invoice" (when completed or requires_invoicing) |
| Handler | `handleCreateInvoice()` → `navigate("add-invoice", { jobId, quoteId?, clientId })` |
| Data flow | Invoice form pre-fills from job; createInvoice stores `jobId`, `quoteId` |
| Fields copied | clientId, jobId, quoteId; line items from `job.pricing.lineItems`; title from job |
| Process | Most automated: job pre-fills invoice form; user can adjust tax, due date, terms |
| Back-link | Invoice stores jobId; no field on Job for "invoiced" or invoice IDs |

---

## 13. Linking to Clients, Properties, Trees, Zones, and Services

### Client
- **Types**: All workflow entities have `clientId`
- **Forms**: SearchableSelect with `addOption` → add-client
- **Firestore**: Stored on all entities
- **List/Detail**: Client name displayed; link to view-client

### Property
- **Request**: propertyId optional
- **Quote, Job**: propertyId required
- **Invoice**: No propertyId; derived from job
- **Forms**: SearchableSelect filtered by clientId; addOption → add-property

### Trees
- **Request**: `requestItems[].treeId`; per-item `serviceTypes`
- **Quote**: `QuoteLineItem.treeIds?` – array per line item
- **Job**: `JobLineItem.treeIds?`; `Job.treeIds` aggregated from line items
- **Invoice**: `InvoiceLineItem.treeIds?` – copied from job; not editable
- **Map**: Request form and view: "Show on map" for trees; `useMapWorkflowTreePick` for map-click to add tree
- **View tree (Teams)**: Related requests/quotes/jobs/invoices for a tree are **derived** by filtering property-scoped (and client-narrowed) lists with `lib/workflow/tree-workflow-links.ts` — not from `Tree.business.activeJobIds` (deprecated field, not written by the app). Invoices match line-item `treeIds` first, else `invoice.jobId` in the set of jobs that already reference the tree (same property-scoped job list). **URL prefill**: `add-request`, `add-quote`, and `add-job` accept optional `treeId` to preselect the tree / line items.

### Zones
- **Request**: `requestItems[].zoneId`; per-item `serviceTypes`
- **Quote, Job, Invoice**: No zone-level linkage in types or forms

### Services
- **Request**: `serviceDetails` (flat multi-select) and `requestItems[].serviceTypes` (per tree/zone)
- **Quote/Job/Invoice**: `serviceType` on line items (string); job has `type` (labor, material, etc.)
- No shared service catalog; free-text / enum in forms

### Map integration
- Request form: trees and zones with "Show on map"; map click adds tree to request
- Request view: same for requestItems
- Job/Quote/Invoice: No map integration in workflow UI

---

## 14. Access Control and Subscription Gating

### Teams-only
- **All workflow drawers** (requests, quotes, jobs, invoices and their view/add/edit) have `visibleForTiers: ['Small Team', 'Mid Team', 'Large Team', 'Enterprise']`
- **Hook**: `useTeamsOnlyAccess()` in each workflow drawer
- **Behavior**: When `showUpgradePrompt` is true, drawer shows "Service requests/Quotes/Jobs/Invoices are available on Teams plans" + link to profile (Upgrade to Teams)
- **Enforcement**: Tier from `getEffectiveSubscriptionTier(userDoc)`; `isTierInGroup(tier, "teams")` for access

### Upgrade CTA
- Links to `navigate("profile")` for subscription management
- No dedicated upgrade-to-teams drawer for workflow; `upgrade-to-teams` exists for general use

### Role-based
- Firestore rules use `organizationId` and team membership; no role distinction within workflow (all team members with access see same UI)

---

## 15. Routing and Navigation

### URL patterns
- `?drawer=requests`
- `?drawer=view-request&requestId=xxx`
- `?drawer=add-request`
- `?drawer=edit-request&requestId=xxx`
- Similar for quotes, jobs, invoices

### Navigation
- `navigate(drawerId, params)` from `useDrawer()` updates URL and navigation store
- `buildDrawerUrl(drawerId, params)` used by sidebar, bottom nav, and external links
- `parseDrawerParams(searchParams, isDrawerRegistered)` extracts drawerId and params

### Entity ID flow
- List → view: `navigate("view-request", { requestId: req.id })`
- View → edit: `navigate("edit-request", { requestId })`
- Add → view: after create, `navigate("view-request", { requestId: newRequest.id })`
- Conversion: params carry requestId, quoteId, jobId, clientId, propertyId as needed

### returnTo
- `addOption` in SearchableSelect uses `returnTo: drawerId` when navigating to add-client/add-property; **returnTo is passed but not confirmed to be consumed** for post-create navigation

---

## 16. Cross-Feature Integration

### Dashboard
- **Jobs this week**: Teams users see `jobsThisWeek` (filtered by schedule); list links to view-job
- **Link**: `navigate("jobs")`, `navigate("view-job", { jobId })`
- **Source**: `useJobs(organizationId)`; date filter in `getDashboardUpcoming` or similar logic

### Weather
- No direct workflow integration in scanned code
- Weather impacts trees; no request/quote/job linkage to weather

### Search
- Global search exists (`drawers/search.tsx`); workflow entities not confirmed to be in search index

### Profile / Subscription
- Workflow gated by `useTeamsOnlyAccess`; tier from userDoc

### Map
- Request: tree/zone selection and "Show on map"
- Job/Quote/Invoice: no map integration

### Tree detail
- **TreeOperationsTab** (`useWorkflowForTree`): lists requests, quotes, jobs, and invoices that reference the tree (see §13 Trees), row-capped with “View all” to list drawers; **Create workflow** popover passes `clientId`, `propertyId`, `treeId` into add-request / add-quote / add-job (add-invoice: client + property only). Property-less trees show a single callout to assign a property before workflow.

### Team / Organization
- All queries scoped by organizationId
- Workflow settings (tax, currency, templates) from team document

---

## 17. Current UX Patterns

### List layouts
- Search bar + filters (status, client) + Add button in header
- Card list: title, status, metadata (client, date, amount), clickable
- Request list: richer cards with date, time, assignees

### Detail layouts
- Header: title, status, optional subtext (number, status)
- Scrollable body with section cards
- Sticky footer with Edit and contextual actions (Mark as, Convert to, Create Invoice, etc.)

### Filters
- Status: Select dropdown
- Client: SearchableSelect with "All clients"

### Patterns
- Table not used; card list throughout
- Mobile: same layout; drawer behavior from navigation store
- Empty states: centered message + optional CTA
- Create: form → submit → navigate to view
- Edit: same form component with entity ID

---

## 18. Gaps, Inconsistencies, and Technical Debt

### Conversion
- `convertedToQuoteId`, `convertedToJobId` on Request never written
- `convertedToJobId` on Quote never written; UI assumes it for "already converted" check
- No line item transfer Quote → Job; user re-enters
- Request → Invoice is misleading (invoice requires job; only clientId passed)

### Invoice
- No edit-invoice drawer; no Edit in view-invoice
- Line items in invoice form are read-only; `onLineItemsChange` is no-op

### Status models
- Request status "quoted" vs Quote status "converted" – no automatic sync
- Quote "converted" status never set when job is created

### Duplicate logic
- Format amount, format date helpers repeated across drawers
- Client display via `getClientDisplayName` – consistent

### Type vs UI
- Invoice `lineItems` type allows editing; UI makes them read-only
- `convertedTo*` fields in types unused in writes

### Unused fields
- `request.requestedDate`, `flexibility` – in type; form has `scheduleStartDate` etc.; requestedDate not clearly used

### Map linkage
- Zones only on Request; not on Quote/Job line items
- Job `treeIds` aggregated from line items; no zone equivalent

### Drawer registration
- edit-invoice not registered; no Edit flow for invoices

### Firestore
- `propertyId` filter on jobs – index may be implicit; verify
- No composite for jobs by propertyId+organizationId if used heavily

### Referential integrity
- No cascade or validation that requestId/quoteId/jobId reference existing documents
- Orphaned references possible if source deleted

---

## 19. Recommended Next Documentation Files

1. **workflow-target-architecture.md** – Desired end-state: atomic conversions, back-links, line item transfer
2. **workflow-conversion-spec.md** – Spec for Request→Quote, Quote→Job, Job→Invoice with field mappings and side effects
3. **service-line-item-model.md** – Unify RequestItem, QuoteLineItem, JobLineItem, InvoiceLineItem; tree/zone linkage
4. **crm-entity-relationships.md** – Client, Property, Request, Quote, Job, Invoice ER diagram and FK rules
5. **invoice-edit-feature-spec.md** – Add edit-invoice, line item editing, payment recording

---

## 20. Appendix

### File inventory (workflow-related)

| File | Purpose |
|------|---------|
| app/drawers/requests.tsx | Request list |
| app/drawers/view-request.tsx | Request detail |
| app/drawers/request-form.tsx | Add/edit request |
| app/drawers/quotes.tsx | Quote list |
| app/drawers/view-quote.tsx | Quote detail |
| app/drawers/quote-form.tsx | Add/edit quote |
| app/drawers/jobs.tsx | Job list |
| app/drawers/view-job.tsx | Job detail |
| app/drawers/job-form.tsx | Add/edit job |
| app/drawers/invoices.tsx | Invoice list |
| app/drawers/view-invoice.tsx | Invoice detail |
| app/drawers/invoice-form.tsx | Add invoice only |
| components/workflow/WorkflowLineItemsEditor.tsx | Shared line item editor |
| hooks/useRequests.ts | Request data |
| hooks/useQuotes.ts | Quote data |
| hooks/useJobs.ts | Job data |
| hooks/useInvoices.ts | Invoice data |
| hooks/useTeamsOnlyAccess.ts | Teams gating |
| hooks/useWorkflowSettings.ts | Tax, currency, templates |
| lib/drawers.ts | Drawer registration |
| lib/drawer-registry.ts | Metadata, tier filtering |
| lib/drawer-utils.ts | URL helpers |
| lib/drawer-context.tsx | navigate, params |
| lib/firebase/firestore.ts | CRUD for requests, quotes, jobs, invoices |
| types/request.ts | Request types |
| types/quote.ts | Quote types |
| types/job.ts | Job types |
| types/invoice.ts | Invoice types |
| firebase/firestore.rules | Security rules |
| firebase/firestore.indexes.json | Composite indexes |

### Key code snippets (summary)

- **Conversion (view-request)**: `getConvertParams()` returns `{ requestId, clientId?, propertyId? }`; handlers call `navigate("add-quote"|"add-job"|"add-invoice", params)`.
- **Quote accept (view-quote)**: `updateQuoteMutation.mutateAsync({ quoteId, data: { status: "accepted" } })` then `navigate("add-job", { quoteId, requestId?, clientId, propertyId })`.
- **Job complete (view-job)**: `updateJobMutation.mutateAsync({ jobId, data: { status: "requires_invoicing", completedAt } })`.
- **Invoice form**: `WorkflowLineItemsEditor readOnly onLineItemsChange={() => {}}` – line items from job, not editable.

### Glossary

| Term | Meaning in codebase |
|------|---------------------|
| Request | Lead/service request; first stage |
| Quote | Estimation with line items and pricing |
| Job | Scheduled work with pricing |
| Invoice | Billing document tied to job |
| Convert | Navigate to add-* with params; no atomic write to source |
| requestId | Optional FK from Quote/Job to Request |
| quoteId | Optional FK from Job/Invoice to Quote |
| jobId | Required FK from Invoice to Job |
| WorkflowLineItemsEditor | Shared component for quote/job/invoice line items |
| Teams-only | visibleForTiers: Small Team, Mid Team, Large Team, Enterprise |
