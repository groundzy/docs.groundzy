# Workflow status redesign — Request, Quote, Job & Invoice

Readable spec for a **simpler status model** and **context-specific actions** per status. **Stored enums and derived helpers** are implemented in the app (see appendix); **full contextual menus** per status may follow in a later ship.

---

## Request statuses

| Status | Meaning (intent) |
|--------|------------------|
| **New** | Just in; not yet scheduled or assessed. |
| **Upcoming** | Scheduled / on the calendar — work or assessment is expected. |
| **Overdue** | Past expected date without completion (operational signal). |
| **Completed** | Work or assessment outcome recorded as done. |
| **Converted** | Linked downstream work exists (quote and/or job). |
| **Archived** | Hidden from active lists; retained for history. |

---

## Request — actions by status

Menus change with status: only show what makes sense.

### Request [New]

| Action |
|--------|
| Schedule Assessment |
| Convert to Quote |
| Convert to Job |
| Archive |
| Print |
| Delete |
| Edit |

---

### Request [Upcoming] · Request [Overdue]

Same menu for both (booking-focused).

| Action |
|--------|
| Text Booking Confirmation |
| Email Booking Confirmation |
| Convert to Quote |
| Convert to Job |
| Print / Download |
| Archive |
| Delete |
| Edit |

---

### Request [Completed]

| Action |
|--------|
| Convert to Quote |
| Convert to Job |
| Print / Download |
| Archive |
| Delete |
| Edit |

---

### Request [Converted] · Request [Archived]

| Action |
|--------|
| View Linked Quote / Job |
| Convert to Quote |
| Convert to Job |
| Print / Download |
| Delete |
| Edit |

> **Note:** No **Archive** on this row — **Converted** can still move to **Archived** via a dedicated control elsewhere if needed; **Archived** is already terminal for “active pipeline.”

---

## Quote statuses

| Status | Meaning (intent) |
|--------|------------------|
| **Draft** | Internal; not yet sent or not awaiting customer. |
| **Awaiting Response** | Sent to customer; waiting on them. |
| **Approved** | Customer accepted terms (ready to convert or schedule). |
| **Changes Requested** | Customer asked for edits before approval. |
| **Converted** | Linked job (or downstream work) created from this quote. |
| **Archived** | Out of active pipeline; kept for records. |
| **Declined** | Customer or business declined; terminal for this sales attempt. |

---

## Quote — actions by status

Same pattern as Request: context-specific menus (send, convert, archive, print, delete, edit).

### Quote [Draft]

| Action |
|--------|
| Preview |
| Send to Customer → moves to **Awaiting Response** |
| Print / Download |
| Archive |
| Delete |
| Edit |

---

### Quote [Awaiting Response]

| Action |
|--------|
| Remind Customer |
| Approve *(records offline / portal acceptance → **Approved**)* |
| Request Changes *(or customer-driven → **Changes Requested**)* |
| Decline *(internal / customer → **Declined**)* |
| Print / Download |
| Archive |
| Edit *(line items: product decides lock vs editable)* |
| Delete |

---

### Quote [Approved]

| Action |
|--------|
| Convert to Job |
| Print / Download |
| Archive |
| Edit *(optional: pricing locked; allow notes only)* |
| Delete |

---

### Quote [Changes Requested]

| Action |
|--------|
| Edit |
| Resend *(after edits → back to **Awaiting Response**)* |
| Print / Download |
| Archive |
| Decline |
| Delete |

---

### Quote [Converted]

| Action |
|--------|
| View Linked Job |
| Print / Download |
| Archive |
| Edit *(read-only or notes-only)* |
| Delete *(discouraged if job exists; confirm destructive)* |

---

### Quote [Archived]

| Action |
|--------|
| View Linked Job *(if present)* |
| Restore *(optional → **Draft** or **Awaiting Response**)* |
| Print / Download |
| Delete |

---

### Quote [Declined]

| Action |
|--------|
| Reopen as Draft *(optional — new sales cycle)* |
| Print / Download |
| Archive |
| Delete |

---

## Job statuses

| Status | Meaning (intent) |
|--------|------------------|
| **Upcoming** | Scheduled for a **future** date (not today) — on the calendar, not yet due. |
| **Today** | Scheduled **for today** — crew focus / day-of list. |
| **Overdue** | Scheduled date has passed; work not completed or not closed out. |
| **Unscheduled** | No date on the calendar yet (still TBD). |
| **Action Required** | Blocked or needs a decision (access, scope, PO, weather call, etc.). |
| **Requires Invoicing** | Work is done; billing must still happen (AR handoff). |
| **Archived** | Filed; hidden from default operational lists. |

> **Note:** The list originally included **Upcoming** twice; this spec uses a **single** **Upcoming** status. If product also wants an explicit **In progress** / **Completed** row, add it — these seven are **schedule- and billing-centric** rather than field-execution granular.

---

## Job — actions by status

### Job [Upcoming]

| Action |
|--------|
| Move to Today *(if job is actually today — timezone-aware)* |
| Text / Email crew or customer |
| Reschedule |
| Mark Overdue *(or rely on daily roll — see transitions)* |
| Move to Unscheduled *(clear date)* |
| Flag Action Required *(→ **Action Required**)* |
| Print / Download |
| Archive |
| Edit |

---

### Job [Today]

| Action |
|--------|
| Text / Email crew or customer |
| Mark complete / Close out *(→ **Requires Invoicing** when work is done)* |
| Reschedule *(→ **Upcoming** or another day)* |
| Did not complete → Overdue *(→ **Overdue**)* |
| Flag Action Required |
| Print / Download |
| Archive |
| Edit |

---

### Job [Overdue]

| Action |
|--------|
| Reschedule *(→ **Upcoming** / **Today**)* |
| Close out / Complete *(→ **Requires Invoicing**)* |
| Flag Action Required |
| Print / Download |
| Archive |
| Edit |

---

### Job [Unscheduled]

| Action |
|--------|
| Schedule *(set date → **Upcoming** or **Today**)* |
| Flag Action Required |
| Print / Download |
| Archive |
| Delete |
| Edit |

---

### Job [Action Required]

| Action |
|--------|
| Add note / task |
| Clear blocker *(→ **Upcoming**, **Today**, or **Unscheduled** — user picks)* |
| Escalate *(optional internal)* |
| Print / Download |
| Archive |
| Edit |

---

### Job [Requires Invoicing]

| Action |
|--------|
| Create invoice |
| View linked invoice *(when created)* |
| Mark invoiced *(when invoice exists — often automatic)* |
| Print / Download |
| Archive |
| Edit *(limited)* |

---

### Job [Archived]

| Action |
|--------|
| View linked quote / request *(if present)* |
| View linked invoice *(if present)* |
| Restore *(optional → prior operational status)* |
| Print / Download |
| Delete |

---

## Invoice statuses

| Status | Meaning (intent) |
|--------|------------------|
| **Draft** | Not yet issued to the customer (editable). |
| **Awaiting Payment** | **Issued** to the customer (including **Issue without sending** with `issuedAt`) **or** sent via email/SMS (`sentAt`); full amount still outstanding. |
| **Past Due** | Past payment terms / due date without full payment. |
| **Paid** | Fully collected; closed for AR. |
| **Bad Debt** | Uncollectible — written off or abandoned per policy (not “paid”). |

> **Note:** No separate **Partially paid** in this model; use **Awaiting Payment** or **Past Due** with **amount paid / amount due** on the document, or add a sub-flag if product needs it.

---

## Invoice — actions by status

### Invoice [Draft]

| Action |
|--------|
| Preview |
| **Record payment** *(manual methods and staff card via Stripe Connect when enabled — **Send email** is not required)* *(full payment → **Paid**; partial → **Awaiting Payment** with balance due)* |
| **Issue without sending** *(optional — **Awaiting Payment**; sets `issuedAt`; does **not** set `sentAt` / email delivery)* |
| Send email *(optional — **Awaiting Payment**; sets delivery timestamps per email flow)* |
| Print / Download |
| Delete |
| Edit |

> **Pay-first:** Clients often pay before any invoice is emailed (cash, off-platform transfer, in-person card). Staff can record payment directly from **Draft** without sending.

---

### Invoice [Awaiting Payment]

| Action |
|--------|
| Record payment *(→ **Paid** when balance zero)* |
| Remind |
| Mark Past Due *(or auto when past `dueDate` — see transitions)* |
| Write off to Bad Debt *(policy)* |
| Print / Download |
| Edit *(policy: lock after send)* |

> **Future — client reminders:** When automated payment reminders exist, gate them on **`sentAt`** (email/SMS actually sent) **or** **`issuedAt`** (staff used **Issue invoice (no email)**) so clients are not nudged for invoices that were never issued to them in Groundzy.

---

### Invoice [Past Due]

| Action |
|--------|
| Record payment *(→ **Paid**)* |
| Remind |
| Escalate / collections note |
| Write off to Bad Debt |
| Print / Download |
| Edit *(policy-driven)* |

---

### Invoice [Paid]

| Action |
|--------|
| View linked job |
| Email receipt |
| Print / Download |
| Edit *(read-only or admin-only adjustments)* |

---

### Invoice [Bad Debt]

| Action |
|--------|
| View linked job |
| Print / Download |
| Add internal note |
| Delete *(rare; policy — prefer immutable record)* |

---

## Status transitions (reference)

### Request

| From | To | Typical trigger |
|------|-----|-----------------|
| New | Upcoming | Schedule Assessment |
| New | Overdue | If date passes before scheduling *(or derived rule)* |
| Upcoming | Overdue | Scheduled date is before today without completion |
| * | Completed | User marks complete |
| * | Converted | Quote and/or Job created (or link recorded) |
| * | Archived | Archive action |
| Archived | *(active)* | Restore *(if product allows)* |

**Overdue** may be **computed** (e.g. scheduled date is before today and status is not Completed, Converted, or Archived) with optional manual override.

### Quote

| From | To | Typical trigger |
|------|-----|-----------------|
| Draft | Awaiting Response | Send to Customer |
| Awaiting Response | Approved | Customer accepts |
| Awaiting Response | Changes Requested | Customer asks for edits |
| Awaiting Response | Declined | Customer or business declines |
| Changes Requested | Awaiting Response | Resend after edit |
| Changes Requested | Draft | Withdraw send *(optional)* |
| Approved | Converted | Convert to Job |
| * | Archived | Archive action |
| Declined | Draft | Reopen as Draft *(optional)* |

### Job

| From | To | Typical trigger |
|------|-----|-----------------|
| Unscheduled | Upcoming / Today | Schedule / book date |
| Upcoming | Today | Calendar rolls to “today” or user moves |
| Upcoming | Overdue | Date passed without closeout *(often daily job)* |
| Today | Overdue | End of day without complete *(or manual)* |
| Today | Requires Invoicing | Work done; ready to bill |
| Overdue | Requires Invoicing | Work done |
| * | Action Required | Blocker flagged |
| Action Required | Upcoming / Today / Unscheduled | Blocker cleared |
| * | Archived | Archive |
| Archived | *(active)* | Restore *(optional)* |

**Today** vs **Upcoming** can be **derived** from `schedule.startDate` + org timezone instead of manual status churn.

### Invoice

| From | To | Typical trigger |
|------|-----|-----------------|
| Draft | Awaiting Payment | Send / issue invoice |
| Awaiting Payment | Past Due | Past `dueDate` without full payment *(often derived)* |
| Awaiting Payment | Paid | Record full payment |
| Past Due | Paid | Payment received |
| Awaiting Payment / Past Due | Bad Debt | Write-off approved |
| Bad Debt | *(none)* | Terminal *(unless reversal policy)* |

**Paid** should set any linked **job** `paidAt` (or equivalent) for reporting — same idea as current `updateInvoice` behavior.

---

## Cross-cutting rules

| Topic | Guidance |
|--------|----------|
| **Print vs Print / Download** | Pick one label app-wide for consistency. |
| **Archive** | Soft-hide from default lists; filters can show Archived. |
| **Delete** | Hard delete or soft-delete per Firestore rules; confirm for Converted / Linked entities. |
| **Edit** | Per status: full edit (Draft), limited (Approved / Sent), read-only (Converted / Paid) — enforce in UI + rules. |
| **View Linked …** | Request: Quote and/or Job. Quote: Job. Job: Quote, Request, Invoice. Invoice: Job (and Quote when present). |
| **Partial payments** | Not a separate invoice status here — track on the document; optionally add a flag later. |

---

## Design notes

1. **Job** statuses emphasize **when** work sits on the calendar (**Upcoming** / **Today** / **Overdue** / **Unscheduled**) plus **Action Required**, **Requires Invoicing**, and **Archived** — not a separate “in progress” row unless product adds it.
2. **Request [Overdue]** vs **Job [Overdue]** — same word; disambiguate in UI (e.g. “Request overdue” vs “Job overdue”) or use icons / column context.
3. **Overdue** on Request and **Past Due** on Invoice are ideally **derived** from dates where possible; keep rules explicit so lists and badges match.
4. **Converted** on Request aligns with **View Linked Quote / Job**; show one or two links depending on what was created.
5. **Quote Declined vs Archived** — Declined is outcome; Archived is filing. A declined quote may still be **Archived** without changing Declined, or product may collapse to one terminal state.
6. **Pipeline alignment** — **Approved quote → Job**, job **Requires Invoicing → Invoice**, **Paid** invoice → job paid signal; **Bad Debt** is explicit uncollectible state (replaces informal void/write-off).

---

## Migration from current system (summary)

Current types use different labels (`types/request.ts`, `types/quote.ts`, `types/job.ts`, `types/invoice.ts`). Before shipping:

| Step | Task |
|------|------|
| 1 | Map each **old** status → **new** status (one-to-one or many-to-one) per entity. |
| 2 | Backfill Firestore documents; feature-flag new enums in UI. |
| 3 | Update indexes, filters, and Firestore rules for new strings. |
| 4 | Add cron or client logic if **Overdue** (request, invoice) or job “late” badges are derived. |
| 5 | QA: every action in the tables above has a defined handler and permission. |

*Detailed mapping table belongs in a migration ticket, not duplicated here.*

---

## Implementation checklist (engineering)

- [ ] Replace / extend `RequestStatus`, `QuoteStatus`, `JobStatus`, and `InvoiceStatus` enums.
- [ ] Implement status-specific action menus (drawer footers / overflow menus) for all four entities.
- [ ] Wire **Schedule Assessment**, booking confirmations, **Send to Customer**, **Remind**, **Convert to Job**, **Create invoice**, **Record payment**, **Void**, etc., to existing or new APIs.
- [ ] Enforce transition rules server-side where security-critical (especially payments and voids).
- [ ] Update list filters, badges, i18n keys, and empty states.
- [ ] Document homeowner / portal behavior if it sets **Awaiting Response** / **Approved** / **Changes Requested**.
- [ ] Align **job `paidAt`** (or successor) when invoice reaches **Paid**; align **primary invoice** / billing handoff when job leaves **Requires Invoicing** (or equivalent).

---

## Appendix: Stored vs derived (v3 implementation)

**Firestore `status` fields** use **snake_case** tokens only (durable business states). **Date-relative** labels from the tables above—**Today**, **Upcoming**, **Overdue** (job/request), **Past due** (invoice)—are **not** written back to `status` when the calendar rolls. They are computed in the client (`app/lib/workflow/workflow-status.ts`) for badges, filters (separate chips from the stored-status dropdown), and dashboards.

| Entity | Stored (`status` field) | Derived (TypeScript only) |
|--------|-------------------------|---------------------------|
| Job | `unscheduled`, `scheduled`, `action_required`, `requires_invoicing`, `archived` | `today`, `upcoming`, `overdue` from schedule + “now” |
| Request | Pipeline tokens (e.g. `new` … `archived`; see `types/request.ts`) | `upcoming`, `overdue` from preferred/requested dates |
| Quote | `draft`, `awaiting_response`, `approved`, `changes_requested`, `converted`, `archived`, `declined` | *(none required for Ship 1)* |
| Invoice | `draft`, `awaiting_payment`, `paid`, `bad_debt` | `past_due` from `dueDate` + `awaiting_payment` |

**Ship 1** (implemented in app): new enums, projections, list filters (**stored** dropdown + **derived** chips where applicable), transitions, i18n for status labels, core actions (convert, invoice, mark paid, archive).

**Ship 2** (contextual drawer menus): The spec tables are exhaustive; the shipped UI uses a **small primary row**, optional **toolbar** (e.g. delete/edit), and a **More** overflow menu for the long tail.

**Action registry (implementation source of truth):** [`app/lib/workflow/workflow-action-registry.ts`](../../app/lib/workflow/workflow-action-registry.ts) exports `getInvoiceDrawerActions`, `getQuoteDrawerActions`, `getJobDrawerActions`, and `getRequestDrawerActions`. Each returns structured `WorkflowActionDef` lists (toolbar / primary / overflow) with `id`, `labelKey`, `icon`, `tone`, `visibility` (`show` | `hide` | `disabled`), and optional `disabledHintKey` / `enabledHintKey` for tooltips. Edit affordances are aligned with **form mode** via the same status → mode helpers as [`workflow-form-mode-policy.ts`](../../app/lib/workflow/workflow-form-mode-policy.ts) (read-only disables Edit; partial modes keep Edit enabled with hints).

**Context:** [`app/lib/workflow/workflow-action-context.ts`](../../app/lib/workflow/workflow-action-context.ts) builds per-entity context (`buildInvoiceActionContext`, `buildQuoteActionContext`, etc.) with predicates such as `isInvoiceEditable` and `isRequestPipelineOpen`. View drawers call the registry once per render and map `actionId` strings to `navigate`, mutations, or stubs.

**Shared footer UI:** [`app/drawers/components/WorkflowDrawerActions.tsx`](../../app/drawers/components/WorkflowDrawerActions.tsx) lays out the toolbar grid (optional slot after the first cell for Request Mark-as), primary buttons, optional slots (e.g. Quote Mark-as, Request Convert), and the overflow **More** menu. In development, a console warning fires if the total visible slots exceed the **≤5–6** target (guardrail 1).

**List row context menus:** CRM list drawers (`requests.tsx`, `quotes.tsx`, `jobs.tsx`, `invoices.tsx`) build row menus from the same registry definitions via [`app/lib/workflow/workflow-list-row-actions.ts`](../../app/lib/workflow/workflow-list-row-actions.ts): flatten toolbar + primary + overflow, apply the same visibility rules as the drawer, prepend **Open** (view entity), and for invoices append **Delete** (list-only; not in the drawer registry). Handlers mirror view drawers (`navigate`, confirm delete, `useUpdateJob` / `useUpdateInvoice` where applicable, stubs → toast).

---

## Related (current system)

Authoritative **as-built** analysis: `docs/features/workflow-request-quote-job-invoice-status-analysis.md`.

---

*End of spec.*
