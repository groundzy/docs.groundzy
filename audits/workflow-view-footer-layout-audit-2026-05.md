# Workflow view footer audit — clients, properties, requests, quotes, jobs, invoices

**Date:** 2026-05-06  
**Scope:** In-drawer **view** mode footers (`DrawerFooter`) for the entities above.  
**Goal:** Document current inconsistency against product rules — **exactly two footer rows**, unified **More** menu (trigger + list items + icons), and a clear split between **primary CTA**, **important visible actions**, and **overflow**.

**Implementation references** (sibling `app` repo, paths from repo root):

- CRM views: `app/drawers/view-client.tsx`, `app/drawers/view-property.tsx`
- Pipeline views: `app/drawers/view-request.tsx`, `app/drawers/view-quote.tsx`, `app/drawers/view-job.tsx`, `app/drawers/view-invoice.tsx`
- Shared footer layout: `app/drawers/components/WorkflowDrawerActions.tsx`
- Action plans: `lib/workflow/workflow-action-registry.ts`

---

## Rule 1: Two rows only

### Current state

| Surface | Rows (typical) | Notes |
|--------|----------------|--------|
| **View client** | 1 or 2 | `grid grid-cols-3`: Delete / Edit / Show on map. If **Teams workflow**, a **second full-width row** is a “Create workflow” popover. Solo tier: **one row** only. |
| **View property** | Same as client | Identical footer pattern. |
| **View request** | **3+** | `WorkflowDrawerActions`: (1) toolbar row — Edit (icon), **More** (icon-only when Mark-as present), Mark-as popover, Delete (icon). (2–N) **each `primary` action is its own full-width row** — e.g. View quote *and* View job → **two stacked rows**. (N+1) **Convert to** popover is **`slotAfterPrimary`** → **another full-width row**. |
| **View quote** | **2–3+** | Toolbar row + **one or more** full-width primary rows (e.g. View job / Convert to job) depending on state. |
| **View job** | **2–3+** | Toolbar + possibly **multiple** stacked primaries from registry (work mode / complete / invoice). |
| **View invoice** | **2** | Toolbar (Edit + **More** fills flex) + one primary (Record payment) when applicable — closest to two rows when only one primary exists. |

**Verdict:** Only client/property (solo) and some invoice states approximate “two rows.” Request/job/quote frequently exceed two rows because **each primary `WorkflowActionDef` renders as a separate block** and **Convert / Mark-as** add full-width controls.

**Root cause (pipeline):** `WorkflowDrawerActions` maps `primaryVis` with one `<WorkflowDrawerActionButton>` **per** definition (`flex flex-col gap-2`), and view-request adds **`slotAfterPrimary`** for Convert.

---

## Rule 2: More menu — same style and icons

### Overflow (“More”) trigger

Implemented in `OverflowMenu` inside `WorkflowDrawerActions`:

- **`footer`:** full-width `h-12`, **icon + “More” label** (`common.more`).
- **`toolbarFill`:** full-width in middle of toolbar when there is **no** `toolbarAfterFirst` (e.g. invoice).
- **`toolbarCompact`:** **icon-only** `h-12 w-12` when `toolbarAfterFirst` exists (request/quote/job share Mark-as with compact More).

So the **same component** already has **two visual treatments** (label vs icon-only) by design — this violates a strict reading of “all more menus same style.”

### Overflow menu list items

- Pipeline overflow: `PopoverContent` → `Button variant="ghost"` rows with **Lucide `Icon` left + translated label** from `WorkflowActionDef`.
- **Not** the same markup as client/property **Create workflow** popover, which uses **native `<button>`** elements with the same `workflow-step-menu` wrapper but different classes and **`workflowStepIconVars` + `WORKFLOW_PIPELINE_ICONS`** tiles.

### Delete / destructive

| Surface | Delete presentation | Consistent? |
|---------|---------------------|------------|
| Client / property | **Icon + visible “Delete” text**, red destructive outline, `size="sm"` in **3-column grid** | Differs from pipeline |
| Request / quote / job | **Icon-only** in toolbar, `h-12`, pinned **after** Mark-as flex region, destructive outline | Differs from CRM |

---

## Mark-as and Convert popovers (pipeline-only)

These are **not** the overflow menu but sit in the **same toolbar row** or **after primaries**, and they **differ across entities**:

| Drawer | Mark-as trigger | Leading icon |
|--------|-----------------|--------------|
| **Request** | `Button variant="outline"` | `CheckCircle` |
| **Quote** | `Button variant="main"` | `CheckCircle` |
| **Job** | `Button variant="outline"` | `WORKFLOW_PIPELINE_ICONS.jobs` |

**Convert to** (request only): default (filled) `Button` with pipeline requests icon — another distinct “primary-like” strip.

Popover list rows generally reuse `workflow-step-menu` + ghost buttons + `WORKFLOW_STEP_ICON_WRAP`; quote/job/request differ slightly in alignment (`align="end"` vs `center`) and width constants.

---

## CRM vs pipeline — structural mismatch

| Aspect | Client / property | Request / quote / job / invoice |
|--------|-------------------|----------------------------------|
| Layout | CSS **grid** `grid-cols-3 gap-2` | **Flex column** `gap-2` |
| Button size | **`size="sm"`** | Toolbar icons **`h-12`**, primaries **`h-12`** full width |
| Edit | Icon + label | Icon-only toolbar |
| Delete | Icon + label | Icon-only toolbar |
| “Workflow” actions | **Popover** “Create workflow” (teams) | Registry-driven toolbar + overflow + status popovers |

Users perceive **different density, rhythm, and affordances** between “CRM detail” and “workflow detail” footers.

---

## Registry-driven actions (summary)

Source: `get*DrawerActions` in `lib/workflow/workflow-action-registry.ts`.

- **Invoice:** Toolbar: Edit. Primary: Record payment (when allowed). Overflow: send email, send text (stub).
- **Quote:** Toolbar: Delete, Edit. Primary: View job or Convert to job (state-dependent). Overflow: send email/text. Mark-as popover when accept/decline allowed.
- **Job:** Toolbar: Delete, Edit. Primary: Work mode **or** Mark complete **or** none; plus Create invoice **or** View invoice — **multiple entries can stack**. Overflow: print/send work order. Mark-as when options exist.
- **Request:** Toolbar: Edit, Delete. Primary: View quote / View job (0–2). Overflow: confirmation email/text. Mark-as when not read-only. Convert popover when `canConvert`.

**Note:** Dev guardrails in `WorkflowDrawerActions` warn if **more than one `tone: "main"`** is visible. Job plan can **schedule two `main`** actions in some contexts (`job.work_mode` and `job.create_invoice`); that should be treated as a **product/layout bug** if it appears in the wild.

---

## Proposed target model (for a follow-up implementation)

This section records **recommended information architecture** to satisfy the two-row rule and unify overflow — not shipped yet.

### Row A — **Context strip** (single horizontal band)

Fixed height (match `h-12` pipeline), one row:

- **Leading:** low-friction **icon tools** (Edit, optional Show on map / future actions) in a consistent order.
- **Middle:** optional **segmented control or single dropdown** for status (Mark-as) **or** convert — **one** control, not multiple full-width stacks.
- **Trailing:** **destructive** as **icon-only** (align with pipeline) *or* labeled — **pick one product-wide**.
- **Overflow:** **one** More trigger — **same visual** everywhere (recommend: **icon-only** with tooltip/`aria-label` + optional badge count; or **always** icon + “More” in a fixed-width slot so the strip never jumps).

### Row B — **Outcome strip** (primary workflow outcome)

- **Exactly one** filled / `main` **primary CTA** per view (e.g. Record payment, Convert to job, Enter work mode).
- **Secondary** navigation (View quote, View job, View invoice) should be **links in content** or **chips**, or collapsed into **one** “Next steps” / overflow group — not **N** full-width buttons.

### What belongs in **More**

Align with existing overflow families:

- **Comms:** email / SMS (stubs included).
- **Documents / delivery** where applicable (invoice send, job work order send/print).
- **Destructive** optional: if not icon in strip, Delete lives here with confirmation on use.
- **Advanced / infrequent:** resend, duplicate (if added), exports.

Use **one** list component: same `Button`/row pattern, same icon sizing, same `workflowStepIconVars` where pipeline icons apply.

### CRM alignment

- Reimplement client/property footer using the **same two-row shell** as pipeline (or extract `WorkflowEntityFooter` used by all six).
- Map **Create workflow** to **Row B** primary or to a **consistent** dropdown in Row A (not a third grid row).

---

## Checklist for closure

- [x] **Two rows max** for all six views, including teams client/property and request convert + dual links.
- [x] **Single** overflow trigger style (document choice: icon-only vs icon+label).
- [ ] **Overflow list** shares one component with create-workflow / mark-as item rows (icon + label + hover). _(Aligned styling; create-workflow still uses native `<button>` rows.)_
- [x] **Delete** — one convention (icon-only toolbar vs text button).
- [x] **Mark-as** — one `variant` + one leading icon policy per entity type or globally.
- [x] **At most one** `tone: "main"` visible per footer (fix job stacking if needed).
- [x] Update [`drawer-visual-inventory.md`](./drawer-visual-inventory.md) or shell docs if footer becomes a shared primitive.

---

## Related docs

- [`workflow-request-quote-job-invoice-audit-2026-04.md`](./workflow-request-quote-job-invoice-audit-2026-04.md) — data/workflow pipeline
- [`drawer-visual-inventory.md`](./drawer-visual-inventory.md) — drawer UI patterns
- App rule: workflow step colors — `.cursor/rules/workflow-step-colors.mdc` (icon tiles for pipeline)
