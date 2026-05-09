# Groundzy — Product & Feature Marketing Reference

This document translates **in-app capability** (as implemented in the Groundzy `/app` repository) into **marketing-ready narrative**: positioning, user outcomes, and **tier-sensitive** framing. Use it for landing pages, sales collateral, release notes, and partner summaries. When external promises are required (SLA, legal, pricing), align with live pricing pages and legal terms—the codebase evolves independently of this file.

---

## Audience snapshot

- **Field crews & arborists** need one place to move from lead → money without retyping scope, losing photos, or guessing “what’s next.”
- **Office & ops** need branding, approvals, taxes, and payment behavior that match how the business actually works.
- **Home & community users** benefit from the **Groundzy AI Wand**, which makes documenting and learning about their trees effortless and fun. Every tree they record has an evolving care and history log, so they can track changes over time. The **Explore feed** reveals local trees, project highlights, and playful challenges that make discovery as engaging as stewardship.

---

## 1. Complete workflow (commercial pipeline)

### What it is

Groundzy’s **workflow pipeline** is an ordered professional path from **who** you work for through **what** you promised, **what** you executed, and **how** you get paid:

**Clients & properties → Work (unified list) → Requests → Quotes → Jobs → Invoices.**

In the product UI this maps to named drawers with a **color-coded pipeline** in navigation (consistent iconography and semantic colors), so users build muscle memory: CRM foundation first, then intake, pricing, execution, and accounts receivable.

### Why it matters (marketing angles)

- **Continuity**: Line items, media, zones, and trees can flow across documents so scope is not re-keyed at every step.
- **Operational clarity**: Stored statuses drive the life of each document; **schedule- and AR-oriented views** (today, upcoming, overdue, past due) are **derived in the product** from dates and status—so dashboards stay honest without “fake” status values.
- **Permission-aware surfaces**: List and detail experiences respect **organization roles and presets** so field staff see assigned work without exposing the whole business.

### Tier notes (for accurate messaging)

- The **full pipeline** (requests through invoices, workflow list behaviors, work items on the map) is tied in code to **Pro and Team-class subscription tiers**, not the entry consumer tiers alone.
- **CRM-style lists** for clients and properties appear for **Plus and above** in navigation; **full client-record CRM** entitlements align with **Pro+** in the entitlement resolver. Marketing should avoid implying “invoices on Plus” unless product explicitly changes that rule.

### Suggested messaging

> Run the full tree-care deal in one system—from the first service request to a branded invoice—without duct-taping spreadsheets, PDF tools, and a separate map.

---

## 2. Client communication & email

### What it is

Client-facing **documents** (service request, quote, job / work order, invoice) can be **emailed from Groundzy** with staff-controlled **To**, **subject**, and **body**. Organization-level **defaults** reduce repetitive typing:

- **Per-document subject templates** (quote, invoice, job, request) with merge tokens such as company and client context.
- A shared **default body template** with placeholders (e.g. client name, document type, document number).
- **Branded PDF output** ties into document settings (logo, colors, header style, footer copy, optional service address / notes / totals breakdown visibility).

Sending flows integrate with a **document send dialog** pattern: edit recipient and message, send, and (where enabled) **copy a client portal link** for sharing outside email.

Server-side email composition can read the team’s stored **`workflowSettings.documentsEmail`** so outbound messages stay on-brand.

### Why it matters

- **Trust**: Clients receive consistent, professional communications instead of ad hoc Gmail threads.
- **Throughput**: Templates turn repeated explanations into one click, with room to personalize per job.
- **Traceability**: Documents track delivery-oriented timestamps (e.g. last sent) separate from purely internal edits.

### Caveats for honest marketing

- Features that are **explicitly stubbed or “coming soon”** in UI (e.g. alternate channels) should not be claimed as generally available—stick to **email + portal link** patterns that exist today.

### Suggested messaging

> Send quotes and invoices that look like *your* company, with messages that still sound like *you*—once you set templates, every send starts ready to go.

---

## 3. Organization settings

### What it is

**Organization / team settings** consolidate how the business presents itself and how work defaults behave:

- **Organization profile**: name, description, industry, website, **team logo** (used in chrome and documents).
- **Workflow defaults** stored on the team (`workflowSettings`): currency, default tax rate and label, **payment terms presets**, quote validity presets, **general terms & cancellation policies**, optional **document-level fees** (custom percentage line items such as processing or late fees), **line item templates** (including service-type mapping into request flows and quick-pick behavior).
- **Document branding** and **email defaults** (see §2).
- **Quote approval** rules: e.g. typed name vs signature image options and disclaimer copy for client acceptance flows.
- **Client payments (Stripe Connect)** and optional **fleet integration (Bouncie)** appear as org-level sections when the user’s role and plan allow (see §5 and §10).
- **Team operational settings**: invite policy, approvals, default permissions model—paired with **member management** (see §9).

**Pro solo mode** in settings is a deliberate UX: some Pro users act as a **personal organization** without a multi-seat team document; the UI surfaces **profile + workflow defaults** while hiding multi-member admin chrome.

### Why it matters

Settings are the **“rules of the road”** for everyone on the account—fewer errors on tax, fewer embarrassed re-sends with wrong terms, and faster quote creation from reusable line items.

### Suggested messaging

> Set the business once—tax, terms, templates, and PDF identity—and every estimate, work order, and invoice inherits the same standards.

---

## 4. Payment processing & fees

### What it is

Groundzy integrates **Stripe Connect** so client card payments land on the **connected account** representing the service business, not a one-off manual charge flow disconnected from invoices.

**Money movement concepts in code:**

- **PaymentIntents** are created on the **connected account**, with optional **application fees** (platform fee in basis points). The default platform fee comes from environment configuration; teams may have an **override** stored in client payment settings where product allows.
- **Who pays card processing** is a first-class setting: the business can absorb fees, or **pass an uplift** to the client so the charged amount **includes** an estimated processing margin (implemented via a configurable basis-points uplift on the charge amount).
- **Invoices** carry **processing fee** fields when quoting workflow introduces them, and may record **last Stripe PaymentIntent** and **last platform fee amount** for audit.
- **Subscriptions / recurring-style** server paths also apply the same fee resolution pattern where used (subscriptions on the connected account with application fee percent when applicable).

### Why it matters

- **Predictable AR**: Card pay aligns with **amount due** and invoice status (`draft`, `awaiting_payment`, `paid`, `bad_debt`).
- **Transparent economics**: Teams can choose whether **they** or the **client** effectively carries Stripe’s processing cost—a common contracting tension Groundzy encodes explicitly.
- **Platform sustainability**: Application fees are bounded and calculated consistently in cents from basis points.

### Marketing guardrails

- Do not quote exact **fee percentages** in public copy unless they are published in official pricing; the codebase supports **configuration** and **per-team overrides**.
- Always mention that **Stripe onboarding, KYC, and country rules** apply—Groundzy orchestrates Connect; **Stripe remains the merchant-of-record platform** for connected accounts.

### Suggested messaging

> Let clients pay cards on the invoice they already approved—with controls for whether your business or your customer carries processing cost, and a clear audit trail on each payment.

---

## 5. CRM — clients, properties, and the map

### What it is

Groundzy’s **CRM** is **map-derived and document-aware**:

- **Clients** support **individual and business** shapes with **multiple emails**, primary flags, contacts for business accounts, addresses, and tags (including **system-maintained** tags for product behaviors).
- **Properties** anchor **service addresses**, boundaries, and relationships to **trees** and **workflow** objects.
- **Tree records** can be **scoped to the viewer’s organization** versus **homeowner or shared** contexts; “shared CRM” helpers model when a professional sees a **mirrored** client/property for trees shared into their org.
- **Client hub** logic aggregates trees linked by **CRM ids**, property scope, and **workflow line items**—so account views remain coherent even when data comes from multiple entry paths.
- **Map affordances** (search, markers, context menus) understand **CRM linkage** so staff jump from geography to account records quickly.

**Entitlements** (high level):

- **Property-side CRM** depth extends to **Plus-class** tiers and above.
- **Client-record CRM** aligns with **Pro and Team workflow** tiers in the entitlement resolver.
- The **full workflow pipeline** is **Pro+**, while **Plus** should be described as **tree + property-centric** unless product marketing standardizes different language.

### Why it matters

- **One graph**: Trees, parcels, jobs, and invoices stop living in disconnected silos.
- **Scales from solo to team**: Personal pros get CRM without enterprise overhead; **team orgs** share a **single counter and numbering** model for operational entities.

### Suggested messaging

> Your clients and properties aren’t a spreadsheet column—they’re the spine that connects the map, the crew’s work, and the invoice.

---

## 6. Work mode — execution on the job

### What it is

**Work mode** is a **job-scoped execution session** (stored under job work-session subcollections) that crews use **in the field**:

- **Phased UI** (pre-start, live session, review, payoff) walks users through **line-item checkpoints** (pending → completed / adjusted).
- **Time tracking** derives **active vs paused** intervals from session events.
- **Photos and notes** attach to work narratives; uploads use dedicated helpers so media lands consistently.
- **Map integration**: workflow pick stores and map chrome can focus **trees, zones, and line-item context** while work mode is active; list surfaces show **“live” / paused** affordances so dispatchers see session state at a glance.
- Completing a session ties back to **job completion** semantics (including pathways that **require invoicing**); summaries can be compiled into **human-readable “stories”** for downstream reporting or client communication patterns.

**Access control** messages in-product reflect real constraints: e.g. work mode requires appropriate **job status**, **permissions**, and **line items** before starting.

### Why it matters

- **Proof of work**: Checkpoints and photos reduce disputes and supercharge change orders grounded in observed conditions.
- **Crew UX**: Large touch-friendly flows and map linkage respect **actual arboriculture work**, not desk-only software.

### Suggested messaging

> Turn “we were there” into structured evidence—time, photos, and line items tied to the job—not a camera roll no one can find later.

---

## 7. Payment records (invoice ledger)

### What it is

Each invoice maintains a **`payments` array** of structured **payment records**:

- **Amount**, **business payment date**, **method** (`card`, `cash`, `check`, `ach`, `wire`, `other`).
- **Source** distinguishes **Stripe Connect** card settlements from **staff-recorded offline** payments.
- Staff can attach **reference** and **internal notes**; offline entries record **who** created the row and **when**.
- Invoice aggregates (`amountPaid`, `amountDue`) coordinate with **status** transitions toward **paid** or **bad debt** handling.

Card flows use **PaymentIntents**; idempotent server logic treats the **PaymentIntent id** as a **stable payment record key** where applicable to avoid duplicate webhook effects.

### Why it matters

- **Audit-friendly**: Mixed cash-and-card operations—normal for tree care—live in **one ledger** next to the PDF the client saw.
- **Support & accounting**: Source and reference fields help reconcile without database archaeology.

### Suggested messaging

> Whether the client paid online or handed the crew a check, the invoice shows a single, coherent payment story.

---

## 8. Team management & access

### What it is

**Team documents** model multi-seat businesses:

- **Roles**: owner, admin, manager, member, viewer (with signup-time mapping from broader signup roles).
- **Seat limits** and **subscription tier** inform max members and team band behavior.
- **Invites**: invite codes with **create / revoke / regenerate** flows; settings gate whether members may invite and whether invites need approval.
- **Danger zone** patterns support **ownership transfer**, member removal, and leaving a team—guarded by **policy checks** (e.g. admin peer rules).
- **Advanced access (v4 policy model)**: members can carry **job-function presets** (`memberPresetIds`) and sparse **per-action overrides** for fine-grained workflow read/write posture without losing understandable role labels.
- **Workflow read overrides** can be tuned for specific members (merge semantics defined in org preset policy).

Server actions **recompute denormalized org policy** on users for **Firestore security rules**, keeping client UX and enforcement aligned.

### Why it matters

- **Scale without chaos**: Field techs see **assigned work**; owners see **everything**; managers sit between without custom IT projects.
- **Compliance mindset**: Separation of duties (who can onboard payments, who can delete CRM entities) is a first-class design axis in policy modules.

### Suggested messaging

> Add the crew without exposing every dollar and every client—Groundzy matches real-world tree companies where not everyone is a billing admin.

---

## 9. Integrations hub (“Apps”)

### What it is

The **Integrations** drawer is a **catalog + capability-gated** hub, grouped into:

- **Map & fleet**
- **Payments**
- **Coming soon** (roadmap cards)

**Live integrations in catalog code:**

1. **Bouncie fleet** — OAuth-based connection; team-tier capability-gated (`integrations.bouncie_fleet`). When connected, **vehicle positions** can participate in map filter layers for eligible orgs; deep-links land in **organization settings → Bouncie fleet** for management.
2. **Stripe client payments** — capability `integrations.stripe_connect`; CTA routes to **Settings → Organization → Client payments** for Connect onboarding and payment behavior.

**Tier gates**:

- The hub itself requires at least **Pro-class** access in tier utilities.
- **Bouncie** additionally expects **team-tier** membership; the card explains **upgrade to Teams** when locked.

### Why it matters

- **Operational reality**: Fleets and card payouts are not “nice-to-haves” for growing tree companies—they’re how work gets scheduled and cash collected.
- **Discoverability**: A single integrations surface prevents settings sprawl.

### Suggested messaging

> Connect the fleet and the wallet—Groundzy’s integration hub is the launchpad for vehicles on the map and client card payments that match your invoices.

---

## 10. Groundzy Explore & Challenges

### What it is

**Explore** is a **destination drawer** with three tabs:

1. **Feed** — Community-style **tree posts** (photos, captions, species context, engagement counts). Participation is gated by **tree media eligibility** (e.g. sharing expectations around having at least **one tree photo**). Posts tie back to **tree** records where applicable; interactions like comments invalidate feed queries for freshness.
2. **Species** — Discovery browsing with **in-drawer search** (species names, challenge-related discovery).
3. **Challenges** — **Gamified, admin-seeded goals** with:

   - **Types**: personal progress, **seasonal** windows, and **geo-fenced** (“curated park” polygon) goals.
   - **Event-driven evaluation**: backend evaluators listen to **groundzy_events** (e.g. tree created, measurement recorded, inspection recorded) with optional payload filters (e.g. identification source).
   - **Progress documents** per user with **distinct entity counting**, completion timestamps, and **reward content** via **content cards** (Markdown body, hero imagery, optional CTAs, “try this next” hints).
   - **Soft initialization / backfill** callable so existing engaged users **don’t start from zero** on first load—marketing can frame this as “fair credit for trees you already mapped.”

Official **Groundzy-seeded posts** can appear differently in the feed (e.g. reduced interaction affordances) to separate community UGC from curriculum-style content.

### Why it matters

- **Retention & learning**: Challenges turn inventory work into **achievable milestones**, especially for new mappers.
- **Community proof**: The feed makes urban forestry and residential tree care **visible**, which supports word-of-mouth growth.

### Marketing guardrails

- Challenges are **content-dependent**; messaging should emphasize **rotating community goals** rather than guaranteeing specific prizes unless a campaign explicitly exists.
- Geo challenges reference **curated park** data—position as “explore your city’s mapped canopy” rather than promising universal global coverage.

### Suggested messaging

> Map trees for work—then share the canopy, learn species, and pick up challenges that reward curiosity without turning science into homework.

---

## Cross-feature narrative (elevator story)

**Groundzy** ties **geographic tree data**, **commercial workflow**, and **field execution** into one experience. CRM records and the map give context; **requests → quotes → jobs → invoices** move money-ready scope; **Work mode** captures what actually happened; **payments and payment records** close the loop with defensible history; **Explore and Challenges** keep individuals and communities coming back after the job is paid.

---

## Document maintenance

- **Source of truth for behavior**: the `/app` repository (drawer registry, types under `types/`, server routes under `app/api/`, Cloud Functions for challenges).
- **Source of truth for commercial promises**: public **pricing**, **legal**, and **support** commitments.

When shipping major UI changes, update this file **or** replace its sections with links to canonical product marketing CMS—avoid stale tier claims.

---

*Derived from static analysis of the Groundzy `app` repo. Feature names and tiers reflect implementation details as encountered during document authoring.*
