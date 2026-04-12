# Workflow participation & external access (v3)

**Status:** Architecture direction — implementation evolves with the v3 rebuild.  
**Audience:** Product, backend, and client engineers defining workflow visibility, rules, and surfaces.

This document complements [home-plus-to-pro-teams-linking.md](./home-plus-to-pro-teams-linking.md) §9 (tree operational guardrails). That section focuses on **what not to ship** on the consumer tree surface today; this document defines the **target model** for how Home / Plus / Pro users (and external parties) **should** see and act on workflow **as participants**, without pretending they are internal pro-org operators.

---

## 1. Problem statement

Today, much workflow truth is effectively **pro-org–owned** (requests, quotes, jobs, invoices scoped by `organizationId`), while **trees** may be **homeowner-owned**. That split creates:

- Empty or misleading UI when homeowners query **their** org scope for **pro-created** work.
- Pressure to “fix” visibility with **org-scoped list queries**, **cross-org hacks**, or **`work_items`** as a user-facing source of truth — all rejected for v3 participant flows.

The v3 fix is **not** to mount internal ops lists (`WorkflowActiveWorkCard`-style) on the tree for non–team users. It is to model **participation** and **canonical records** explicitly.

---

## 2. Principles

| Principle | Meaning |
|-------------|--------|
| **Event-first** | Meaningful lifecycle changes emit **canonical events** (e.g. quote_sent, job_scheduled, invoice_paid). Long-term, history and projections derive from them. |
| **WorkItems are projections** | `work_items` and similar structures are **derived** for speed/UI — not the participant-facing **system of record**. |
| **Tier is not security** | Subscription tier controls **product features** (what you can create/manage). **Participant relationship** controls **record access** for external/customer paths. |
| **Rules before UI** | Access is defined by **data model + rules**, not ad hoc drawer gates. |

---

## 3. Two separable concerns

| Concern | Question it answers |
|---------|---------------------|
| **Organization ownership** | Who **manages and operates** this work **internally** (pro org, team, assignment, internal status)? |
| **Participant access** | Who may **view / accept / pay / sign / comment** as an **external** or **cross-party** actor on **this** item? |

Internal ops lists answer the first. The global **Work** surface (see §7) and participant item views answer the second.

---

## 4. Two product layers

1. **Participation access (authorization)**  
   Can this principal see or act on **this** workflow item because they are a **participant** with allowed actions?

2. **Product capability (tier)**  
   What **creation and management** tools does the subscription unlock (full internal pipeline vs receive-and-act only)?

Home / Plus users may have **no** internal pipeline creation on **pro** work but **must** be able to **view / accept / pay** when the canonical record says so.

---

## 5. Canonical workflow record (directional shape)

Workflow entities (request, quote, job, invoice) should eventually carry, at minimum:

- **Stable id** and **type** / **status**.
- **Links:** `treeId`, `propertyId`, `clientId`, `orgId` (and line-item tree refs where applicable) — not only org scope.
- **Participants:** structured entries (not only “owner org”):
  - roles (e.g. customer, pro_org, assignee),
  - **permission flags** per participant or per role: `canView`, `canAccept`, `canPay`, `canComment`, … (exact matrix is product + security).
- **Event history** (or parallel **event stream**): created, sent, viewed, accepted, scheduled, completed, invoiced, paid — aligned with event-first v3.

**UI question (target):**  
“What workflow items is this user a **participant** in, and **what actions** are allowed?” — **not:** “What can I read from **this org’s** collection?”

**Firestore schema (participants, denormalized ids, events, indexes):** [Canonical workflow schema (v3)](./canonical-workflow-schema-v3.md).

---

## 6. Two experiences on the same canonical data

### 6.1 Internal workflow workspace (pros / teams)

- Full operational surfaces: lists, assignment, scheduling, internal statuses.
- Evolves as **projections** over canonical workflow + events.

### 6.2 Participant-facing surface (any tier)

- **View**, **accept**, **pay**, status, optional messaging — **action-scoped**.
- **Not** the same drawers as internal invoice/job management unless the user is also an internal operator for that org.

Home / Plus / Pro **must not** see “the org’s full workflow list” as a substitute for participant access unless they **are** internal operators for that org.

---

## 7. Entry points (product navigation)

Without an explicit decision, teams default to **“just show it on the tree”** — which overloads the tree page and confuses **context** with **primary discovery**.

### 7.1 Product name for the global participant surface

This surface will be **top-level nav**, shape onboarding copy, and define mental model — it should not stay fuzzy.

**Architecture decision (name):** Use **Work** as the default product label for the global participant workflow entry (nav item, screen title, and copy). **My Work** is an acceptable alternative if stronger ownership clarity is needed for mixed consumer/pro audiences. **Tasks** is a plausible alternative if the product wants a more action-oriented frame.

**Avoid** using **Inbox** as the primary nav label: it reads as passive and email-like and under-sells actionable workflow (accept, pay, schedule).

In this document, **“Work”** means that global participant list + entry unless otherwise noted.

### 7.2 Options considered

| Option | Description | Risk |
|--------|-------------|------|
| **A — Global only** | A single **Work** (or **My Work**) surface: all participant items, cross-tree. | Tree feels disconnected without drill-down. |
| **B — Tree-first only** | Participant workflow **only** from the tree; no global list. | Weak discovery; many trees; “card on tree” becomes the whole product. |
| **C — Both** | **Global Work primary**; **tree = contextual slice** (events + links, not a second inbox). | Two surfaces + clear nav between them. |

### 7.3 Decision (v3 recommendation): **Option C**

- **Primary:** Global **Work** — all items where the current user is a **participant**, across trees and properties.
- **Secondary:** **Tree-linked context** — see §7.5; same participant-safe **item** views as Work, reached from the tree (not a separate product).

> Participant-facing workflow is accessed via: (1) a global **Work** surface (**primary**), and (2) **contextual entry from the tree** via **events** and optional links (**secondary**). The tree is **not** the sole owner of participant workflow visibility.

**Discovery:** Sidebar, profile, notifications, email/SMS deep links, and tree context must route into the **same** participant experiences — not different ad hoc UIs per entry point.

### 7.4 One canonical participant item view (hard rule)

**Same workflow object → same participant-facing UI everywhere.**

All entry points — **Work**, **tree**, **notifications**, **external links** (token/portal) — must **resolve to a single canonical participant-facing item view** per workflow object (one quote view, one invoice view, one job view, etc., each with action affordances driven by permissions).

This prevents multiple quote UIs, diverging invoice experiences, and long-term fragmentation.

*Internal* operator views (full CRM / team tools) remain separate; this rule applies to **participant-safe** surfaces only.

### 7.5 Tree surface (contextual slice, not sole entry — and not a mini Work)

- The tree remains a strong **anchor for context** (which property, which care relationship).
- The tree page should **not** primarily mount **internal** `WorkflowActiveWorkCard`-style lists for homeowners.
- **On the tree, show (in this order of emphasis):**
  1. **Events (primary):** canonical workflow **events** relevant to that tree (timeline / milestones: quote sent, job scheduled, invoice due, etc.).
  2. **Linked items (secondary, optional):** shortcuts to open the **same** canonical participant item views as **Work** — not a second list that duplicates Work row-for-row.

**Do not** build a **mini inbox on the tree** (full cross-section of participant items scoped to the tree as if it were the global Work surface). The global **Work** surface holds the cross-tree list; the tree holds **event-first** context plus **deep links** into the shared item views from §7.4.

For **`groundzy_events`**, tree projections, and query-friendly participant fields, see [canonical-workflow-schema-v3.md](./canonical-workflow-schema-v3.md).

---

## 8. Participant identity

“Homeowner / customer / pro” labels are insufficient; **identity shape** drives invites, payments, quote acceptance, and onboarding.

| Topic | Decisions to resolve in implementation |
|-------|----------------------------------------|
| **Authenticated user** | Is the customer **always** a Groundzy `uid`, or can participation be **email/phone-first** before a full account exists? |
| **External participants** | Can a participant be **only** an external identifier (e.g. email + token) for portal/magic-link flows? How does this align with quote/invoice **external delivery**? |
| **Pre-signup** | What is visible or actionable **before** signup (tokenized quote, pay link)? How does identity **merge** onto a `uid` after signup? |
| **Pro side** | Pro org and members are usually authenticated; still distinguish **org role** from **participant on a specific item**. |

**Rule of thumb for design:** Distinguish **participant record on the workflow object** from **`users/{uid}` document** so token flows, CRM clients, and shared trees stay consistent.

Until specified in implementation rules, **do not** assume `participant === users/{uid}` only.

---

## 9. Event model (lifecycle)

Every meaningful transition should be representable as (or backed by) **canonical events**, for example:

- quote_sent, quote_viewed, quote_accepted  
- job_scheduled, job_completed  
- invoice_issued, invoice_paid  

Exact naming and storage belong to the v3 event pipeline; this doc only requires **alignment** with event-first v3 and **participant-safe** read surfaces.

---

## 10. Read projections

Allowed as **derived** indexes for speed and UI:

- Pro ops list  
- Participant **Work** list (global)  
- **Tree** event timeline (and optional item deep links — not a duplicate Work list)  
- Billing summary  

They must be **derivable from canonical workflow + events**, not a second source of truth.

---

## 11. What not to do (guardrails)

- Expose **org-scoped workflow lists** to users who are **not** internal operators for that org, as a substitute for **participant** access.
- **Cross-org collection queries** from tree pages to simulate visibility.
- Use **`work_items`** as the **participant-facing** truth layer.
- Make Home / Plus / Pro “kind of Teams” via **ad hoc** UI gates without the participant model.

---

## 12. Migration path (phased)

| Phase | Focus |
|-------|--------|
| **1** | Minimal **tree operational surface** for consumers (copy/CTAs; no fake cross-party workflow lists) — see [home-plus-to-pro-teams-linking.md](./home-plus-to-pro-teams-linking.md) §9. |
| **2** | Canonical workflow entities + **participant model** + **participant identity** rules. |
| **3** | **Participant item views** (canonical per §7.4) + global **Work**; quote accept; invoice pay; request/job status as needed. |
| **4** | **Tree-linked event timeline** (secondary entry); same participant-safe item views as Work — not a mini Work on tree. |
| **5** | Internal ops surfaces as **projections** over the same canonical model. |

---

## 13. Codebase pointers (current app)

- [`WorkflowActiveWorkCard`](../../app/drawers/components/WorkflowActiveWorkCard.tsx) — internal-style list using **org-scoped** hooks; not the v3 participant inbox.
- [`TreeWorkflowSections`](../../app/drawers/view-tree/components/TreeWorkflowSections.tsx) — wires that card into view-tree Ops for **internal** workflow scope.

Future **participant Work** surface and **tree event timeline** should be **new surfaces** backed by canonical/participant reads — not this card bolted onto homeowners without the model above.

---

## 14. Related documentation

| Document | Relevance |
|----------|-----------|
| [canonical-workflow-schema-v3.md](./canonical-workflow-schema-v3.md) | Firestore `v3Workflow`, participants, `groundzy_events`, indexes, migration |
| [home-plus-to-pro-teams-linking.md](./home-plus-to-pro-teams-linking.md) §9 | Consumer tree Ops guardrails; links here |
| [quote-external-delivery-and-homeowner-signup.md](./quote-external-delivery-and-homeowner-signup.md) | External quote delivery, tokens, signup |
| [../features/crm.md](../features/crm.md) | Current org-scoped CRM / workflow |

---

## Document history

| Date | Change |
|------|--------|
| 2026-04-04 | Initial architecture: participation model, entry points (Option C), identity, events, guardrails, migration phases |
| 2026-04-04 | Global surface named **Work**; §7.4 canonical participant item view; tree = events primary + linked items secondary (no mini Work) |
| 2026-04-04 | Links to [canonical-workflow-schema-v3.md](./canonical-workflow-schema-v3.md) (§5, §7.5, §14) |
