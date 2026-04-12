# Plant Health Care (PHC) — Feature Plan

This document is a **planning artifact** moving toward **execution-ready architecture**. It covers structured plant health care: pests, diseases, treatment programs, products, compliance, and reporting—grounded in the **current Groundzy codebase** (Next.js App Router, Firebase/Firestore, drawer-based UI, CRM workflow, tree history).

**Status:** v1.2 — adds execution locks (job↔episode direction, observation↔episode UX, Pro ad hoc products, TreeEvent Phase 1 choice, program constraints, terminology). Remaining risk is **execution discipline** (especially not overbuilding Phase 1). Implementation must not start until §5–§6 and §6.1 are explicitly signed off.

**Related docs:** [trees.md](./trees.md), [crm.md](./crm.md), [map-and-zones.md](./map-and-zones.md), [ai.md](./ai.md), [weather.md](./weather.md), [drawer-system.md](./drawer-system.md).  
**Canonical types:** `types/tree.ts`, `types/job.ts`, `types/zone-service.ts`, `types/species.ts`, `types/quote.ts`.  
**Data access:** `lib/firebase/firestore.ts`, security `firebase/firestore.rules`.

### Terminology (use consistently in docs and code)

| Canonical term | Meaning | Avoid |
|----------------|---------|--------|
| **Application line** | One product application row belonging to a **treatment episode** (rate, unit, product reference or ad hoc snapshot). | “Application record,” vague “lines” without “application” prefix in domain discussions. |
| **Treatment episode** | One dated visit / operational container for a tree (or zone batch). Holds metadata + **application lines**. | Collapsing episode + line into “service.” |
| **Observation** | What was **seen** (pest/disease/disorder instance), independent of whether anything was applied. | Calling observations “treatments.” |

Firestore paths may use short segment names (e.g. subcollection `applications`); TypeScript types should read **`PhcApplicationLine`** / `applicationLine` so intent is unambiguous.

---

## 1. Vision & scope

### 1.1 What “PHC” means here

**Plant Health Care** in arboriculture/landscape contexts usually spans:

- **Diagnostics**: identifying pests, diseases, abiotic disorders, and site stressors.
- **Treatment**: applications (fertilizer, pesticide, fungicide, biostimulants, soil amendments), methods, rates, timing.
- **Programs**: recurring visits, seasonal rotations, IPM (Integrated Pest Management) strategies.
- **Compliance**: jurisdictional rules, applicator licensing, record-keeping, restricted products, buffer zones, REI (re-entry intervals), notification requirements.
- **Outcomes**: follow-up inspections, efficacy notes, customer communication, billing linkage.

Groundzy already models **individual plants (trees)**, **zones**, **CRM workflow**, **service history**, and **species-level care hints**. PHC should **extend** those primitives rather than replace them.

### 1.2 In scope (phased)

| Area | Description |
|------|-------------|
| **Taxonomies** | Curated or org-custom lists of pests, diseases, disorders; link to species and regions. |
| **Observations** | Structured findings on a tree/zone (what, severity, location on plant, photos). |
| **Treatments** | **Treatment episodes** with **application lines** (product, rate, volume, method, target taxon, weather snapshot on episode). |
| **Programs & scheduling** | Recurring PHC visits; alignment with jobs and (where needed) legacy `ServiceRecord` / `ZoneServiceRecord` summaries. |
| **Workflow** | Requests/quotes/jobs line items that **reference** PHC operational records—not define them. |
| **Reporting** | Property/zone summaries, treatment logs, export for audits (after structured capture exists). |
| **AI assistance** | Suggest identification questions, treatment *education* copy, checklist generation (not a substitute for licensed advice). |

### 1.3 Explicit non-goals (initially)

- Replacing professional judgment or regulatory advice; the product remains a **field and office tool**, not a legal authority.
- Full **label database** for every jurisdiction worldwide (consider integrations or phased catalog).
- **Prescription-only** or human-medicine adjacent flows.

### 1.4 v1 product focus: woody plants first

Groundzy’s core identity is **tree- (map item-) centric**. PHC v1 should **optimize for woody plants** (trees/shrubs already modeled as map items). Lawn, turf, annual beds, and broad “all plant care” semantics differ in UX, units, and compliance; folding them in too early **risks a generic data model and crowded UI**.

- **v1:** `plant_context: "woody"` (implicit or explicit on PHC records).
- **Later:** optional expansion via `plant_context` enum and separate presets—not a second product, but a deliberate phase.

---

## 2. Current state in the codebase

### 2.1 What already exists (anchors for PHC)

**Service taxonomy** — `ServiceType` in `types/tree.ts` already includes PHC-adjacent values, for example:

- `FERTILIZATION`, `PEST_CONTROL`, `DISEASE_TREATMENT`, `SOIL_AMENDMENT`, `ROOT_CARE`, `WATERING`, `MULCHING`, plus assessments (`HEALTH_ASSESSMENT`, `RISK_ASSESSMENT`, etc.).

**Per-tree history** — `TreeHistory` supports:

- `ServiceRecord` (with `materials[]`, `equipment[]`, `weather`, `nextService`, photos, quality check).
- `InspectionRecord` (health/structural/risk/comprehensive with component scores and `results[]` sections).

**Event stream (scalability)** — `TreeEvent` / `tree_events/{treeId}/events/{eventId}` is documented in `types/tree.ts` as the long-term home for append-only history (Phase 4+).

**Zone-level bulk care** — `ZoneServiceRecord` (`types/zone-service.ts`, subcollection `zones/{zoneId}/zone_services`) models bulk operations with `serviceType`, `scheduledDate`, status workflow.

**CRM** — Requests, quotes, jobs, invoices (`types/request.ts`, `types/quote.ts`, `types/job.ts`). Quotes use a coarse `serviceType` including `"treatment"`. Job `JobLineItem` supports `treeIds`, `zoneIds`, generic `material` lines.

**Species context** — `SpeciesExtended.care` includes `pestResistance`, `diseaseResistance`, `commonPests`, `commonDiseases` (string lists today).

**UI patterns** — `TreeHealthAssessment` (`components/trees/tree-health-assessment.tsx`), health trend UI (`TreeHealthTrendAndRisk.tsx`), `AddRecordTypePopover` with a **“Treatment”** option that maps to `entryType: "service"` (same path as generic service).

**AI** — `lib/ai-chat/context-builder.ts` injects species care; `lib/ai-chat/suggestions.ts` offers pest/disease prompts (e.g. `howToTreatDisease`).

**i18n** — Keys exist for pest control and wizard copy (`lib/i18n/messages.ts`); some workflow labels remain English literals in places (mixed state).

### 2.2 Gaps (why PHC is “massive”)

| Gap | Impact |
|-----|--------|
| No first-class **pest/disease/product** entities | Cannot filter, report, or link observations to stable IDs. |
| `materials[]` / line items are **unstructured strings** | No rates, EPA #s, units, lot numbers, or SDS links. |
| **Inspection** `results[]` is free-form sections | Hard to aggregate “all ash with EAB” or severity trends. |
| **Zone vs tree** treatment attribution | Bulk zone service does not per-tree product breakdown without extra modeling—**high design risk** (see §7). |
| **Compliance** fields absent | REI, applicator license, wind speed, temperature, notification flags. |
| **Programs** | `nextService` on `ServiceRecord` is a single timestamp, not a full program/cadence model. |
| **Duplication risk** | Without a single source of truth, the same visit may be copied across `tree.history`, PHC stores, jobs, and dashboards (see §6). |

---

## 3. Personas & jobs-to-be-done

| Persona | Primary jobs |
|---------|----------------|
| **Field arborist / technician** | Log observation + treatment quickly; photos; offline-friendly later; see what was done last visit. |
| **PHC manager** | Build programs per property/zone; assign crews; ensure documentation for renewals and audits. |
| **Office / sales** | Quote PHC scopes; convert to jobs; invoice materials and labor separately. |
| **Homeowner / shared viewer** | See high-level plan and safety notices (respect `tree_permissions` / restricted views). |
| **Admin (Groundzy)** | Curate global pest/disease/product reference data; feature flags by region. |

---

## 4. Proposed domain concepts

These are **logical** entities; physical Firestore layout is in §7.

1. **Taxon / condition** — pest, disease, abiotic disorder, or “unknown” with free text.
2. **Observation** — instance of a condition on a tree (or zone aggregate) with severity, notes, media, optional ID from taxon catalog.
3. **Treatment episode** — one visit’s operational container: when/where/who, weather snapshot, links to **application lines** and optionally to a job.
4. **Application line** — one product application: product (library or ad hoc), rate, volume, unit, method, target taxon optional, compliance fields.
5. **Product** — reference row (org library): name, active ingredient(s), type, default unit, regulatory ids (optional), SDS URL, notes.
6. **Program** — named plan with cadence and scopes (later phase); see **§4.1** for constraints so Phase 3+ does not break the episode model.
7. **Recommendation** — suggested follow-up (from inspection, note, or AI) that may spawn a **draft job or draft episode** without requiring a prior observation.

### 4.1 Program model — constraints to set now (even though programs ship later)

Programs stay **conceptual in Phase 1**, but these rules prevent a redesign later:

1. **Programs never replace episodes.** A program generates **instances**: dated **treatment episodes** (and optionally **draft jobs**). Historical truth remains episode + **application lines**.
2. **Recurrence is data on the program document** (e.g. RRULE-like or explicit visit schedule); each executed visit is still one **episode** with `programId?` set—no “virtual” episodes that exist only on the program.
3. **Scopes are references, not copies of tree state:** program links **property / zone / tree IDs** (or explicit membership snapshots only when generating a specific visit). Do not embed full tree inventories that go stale—use IDs + resolve at generation time, or snapshot at visit spawn only.

---

## 5. Entity boundaries — hard rules (freeze before coding)

These rules prevent **inconsistent implementations** across tree history, jobs, invoices, and reporting.

| Rule | Specification |
|------|----------------|
| **Observation independence** | An **observation** may exist **without** any treatment episode or visit. It is a first-class record (e.g. “saw flagging, no treatment yet”). |
| **Episode vs lines** | A **treatment episode** is **one operational visit** (or one dated zone treatment batch). It contains **one or more application lines**. Do not model a line as a full episode. |
| **ServiceRecord relationship** | Legacy/generic `ServiceRecord` may remain for **non-PHC** work or as a **thin summary** pointer (`phcEpisodeId`, `serviceType` for filtering)—but **must not** be the canonical store for structured PHC payloads long term. |
| **Recommendations** | A **recommendation** may create a **draft job** or **draft episode** with **no observation** (e.g. proactive spray program). If an observation exists, link it optionally. |
| **CRM references PHC** | Jobs/quotes/invoices hold **references and rollups** (IDs, totals, line copy for billing)—not the operational definition of applications. **PHC records are operational truth; billing summarizes them.** |
| **Timeline rendering** | `view-tree` (and eventual `tree_events`) **renders** PHC observations and episodes as timeline items; **avoid storing duplicate full PHC payloads** inside `ServiceRecord` once dedicated records exist. |
| **Job ↔ episode (direction)** | **Canonical FK lives on the episode:** `treatmentEpisode.jobId?: string`. A job **does not** own the authoritative list of episodes in the PHC model—**query** episodes where `jobId == thisJob`. Optional denormalized cache on the job document is allowed only if updated in the **same** write transaction/path as `episode.jobId` (never divergent “or” semantics). |

---

## 6. Architecture decisions to lock before implementation (ADR summary)

Implementation is **not execution-safe** until these six choices are **explicitly decided** (defaults below reflect review feedback; replace if product disagrees).

| # | Decision | **Recommended default** |
|---|----------|-------------------------|
| **1** | **Canonical PHC entities and relationships** | **Observation**, **Treatment episode**, **Application line** (children of episode), **Product** (org library), **Program** (later). Recommendations optional → draft job/episode. Taxa as reference catalog. |
| **2** | **Dedicated collections vs embedded extensions** | **Option B (dedicated PHC storage) is the default architecture** for any roadmap with reporting, auditability, product library, or compliance fields. Option A (embed in `ServiceRecord`) is only acceptable for a **time-boxed throwaway prototype**—not for production PHC. |
| **3** | **Zone treatment attribution** | **At minimum:** persist a **membership snapshot** at treatment time (tree IDs—or identifiers—that were in the zone when treated). **Optionally:** explicit **per-tree allocation records** (split rates, partial treatment). **Do not** rely on “current zone membership” alone for historical truth—membership changes break exports and efficacy trends. |
| **4** | **Source of truth vs tree history** | **PHC records are primary.** Tree history / `tree_events` are **projections** for timeline UX (and legacy compatibility), not a second full copy of application payloads. |
| **5** | **Tiering** | **Pro:** structured **observations** and **treatment episodes + application lines** on trees, **view-tree** PHC timeline; product fields may be **ad hoc** (free text) when no library tier. **Teams:** **product library**, **job/invoice linkage**, **programs**, **zone PHC** (snapshot/allocation), **reporting/exports**, compliance-oriented workflows. (Exact gates must match `SubscriptionTier` / `lib/drawer-registry.ts` when built.) |
| **6** | **“Compliance-ready documentation” in v1** | **Narrow slice only** (see §10.1)—not a full org/region configuration framework on day one. |

### 6.1 Execution locks (sign off with §6)

| # | Lock | **Recommended default** |
|---|------|-------------------------|
| **A** | **Episode ↔ job** | **`episode.jobId` only** (canonical). Jobs load linked episodes via query. No independent `job.phcEpisodeIds[]` as a second source of truth unless maintained as strict denormalized cache (same write path). |
| **B** | **TreeEvent in Phase 1** | **No dual-write** to `tree_events` for PHC in Phase 1. Timeline reads **directly** from `phc_observations` / `phc_episodes` (+ nested **application lines**). Add **thin** `TreeEvent` rows that **reference** PHC doc IDs in a later phase when the event pipeline is ready—avoids rework from premature dual-write. |
| **C** | **Pro ad hoc products** | Each **application line** stores `productMode: "library" \| "ad_hoc"`. **Ad hoc** = inline snapshot on the line (`adHocLabel`, optional regulatory text fields)—**no** `phc_products` row on Pro. **Teams:** “**Save to library**” creates a `phc_products` doc and rewires the line to `productMode: "library"` + `productId` (upgrade migration = optional batch promote, not automatic). |

---

## 7. Data model strategy

### 7.1 Guiding principles

- **Dedicated PHC documents** (subcollections under `trees/{treeId}/…` and org-scoped libraries) as the default.
- **Reuse `organizationId` / `databaseCode`** patterns from existing trees and CRM entities (`docs/reference/firestore-collections.md`).
- **Align with `TreeEvent`:** each PHC observation/episode should map to a timeline event that **references** the PHC doc ID, not duplicate nested application data.
- **Zone treatments** always carry **attribution policy data** (§6 item 3).

### 7.2 Option A — Extend existing records (prototype only)

- Optional `phc` blobs or ID arrays on `ServiceRecord` / `InspectionRecord`.

**Use when:** proving UX in under one sprint with **no reporting requirement**.  
**Do not use when:** product library, audits, or multi-year history matter—**migrate off immediately** or incur rework.

### 7.3 Option B — Dedicated PHC collections (**default architecture**)

Illustrative paths (names subject to engineering review):

| Collection path | Purpose |
|-----------------|--------|
| `phc_taxa/{id}` (or parallel catalog pattern) | Global + regional pest/disease reference |
| `phc_products` (with `organizationId`) or `organizations/{orgId}/phc_products/{id}` | Org product library |
| `trees/{treeId}/phc_observations/{id}` | Observations |
| `trees/{treeId}/phc_episodes/{episodeId}` | Treatment episode |
| `trees/{treeId}/phc_episodes/{episodeId}/applications/{lineId}` **or** flat `phc_application_lines` with `episodeId` | **Application lines** (see terminology block) |
| Zone-level episode | `zones/{zoneId}/phc_episodes/{id}` (or org-scoped) with **embedded `treeMembershipSnapshot`** and optional **`allocations[]`** |

**Episode document (illustrative fields):** `jobId?` (canonical link to CRM), `linkedObservationIds?` (or `primaryObservationId?`) for UX flows in §8.2.1, `programId?` when spawned from a program (later).

**Workflow links:** **`phc_episodes.jobId`** → optional link to `jobs/{jobId}`. Jobs discover episodes with `where("jobId", "==", jobId)` (+ `organizationId` as needed). Invoice/job **line items** remain billing summaries; optional `episodeId` on a line item for audit. **Do not** treat two-way loose linking as acceptable (§5, §6.1-A).

### 7.4 TypeScript surface

- New module e.g. `types/phc.ts` with Zod schemas mirroring Firestore.
- Extend `TreeEventType` or `TreeEvent.data` discriminated unions so timeline code can render PHC without parsing generic `ServiceRecord`.

### 7.5 Security rules

- Mirror tree access: if user can edit tree, they can create PHC child docs for that tree (subject to tier).
- **Restrict financial and license fields** to roles (similar to cost masking on `ServiceRecord`).
- Global reference taxa: read for authenticated users; write for global admins only.

### 7.6 Indexes

Plan composite indexes for:

- List episodes by `organizationId` + `completedAt` / `scheduledAt`.
- List observations by `taxonId` + denormalized `propertyId` / `zoneId`.
- Zone exports: by `zoneId` + `episodeDate` with snapshot queries.

---

## 8. Product & UX plan

### 8.1 Navigation model — avoid drawer sprawl

Follow **drawer registration** (`lib/drawers.ts`, `lib/drawer-registry.ts`):

- **Prefer** extending **`view-tree`**, **`view-zone`**, and **`view-job`** for PHC actions and summaries.
- **PHC hub** (org-level) stays **minimal** at first: shortcuts to product library (Teams), open observations, program list (later)—not a second app shell.

| Deliverable | Likely surface |
|-------------|----------------|
| Tree PHC | `view-tree` tab/section + timeline |
| Zone PHC | `view-zone` (extends existing service-type presets) |
| Job linkage | `view-job` / `job-form` panel |
| Product library | dedicated drawer or sub-drawer; **Teams-first** |

### 8.2 View Tree

- **“Health care”** timeline: observations + episodes (**application lines** grouped under each episode), linked jobs (from `episode.jobId`).
- **Quick add:** observation wizard; **Treatment** popover routes to **episode + application lines** flow when PHC is enabled.

### 8.2.1 Canonical observation ↔ episode UX (avoid fragmented drawers)

One **canonical** interaction model for Phase 1 (adjust copy, not data rules):

| User intent | Flow |
|-------------|------|
| **Observation → treat** | From observation detail: **“Log treatment”** opens **new treatment episode** with `linkedObservationIds[]` (or single `primaryObservationId`) pre-filled. |
| **Treat without prior observation** | **New episode** with optional **“Add observation”** inline (creates observation + links in one save) or skip—observation remains optional. |
| **Observation only** | Save observation; no episode required. |
| **Recommendation / office** | **Draft job** (Teams) or **draft episode** (Pro field-first): recommendation creates placeholder; field user completes episode + **application lines**. |

**Rule:** Any screen that creates an episode should offer **link existing observation**, **create inline observation**, or **neither**—all three remain valid per §5.

### 8.3 Forms & validation

- **React Hook Form + Zod** (existing pattern).
- **Unit math:** rate × area/volume; imperial/metric per prefs.

### 8.4 i18n

- All new strings via `useI18n()` / `lib/i18n/messages.ts` (`en`, `es`, `fr`).

### 8.5 Mobile / field

- Short-term: responsive drawers. Long-term: offline queue (separate epic).

---

## 9. Integration matrix

| System | Integration |
|--------|-------------|
| **Tree history / timeline** | **Phase 1:** render PHC from **`phc_*` subcollections** directly in `view-tree`. **No** PHC dual-write to `tree_events` until a later phase (§6.1-B). Avoid duplicating full PHC payloads in `ServiceRecord`. Legacy `ServiceRecord` may remain for non-PHC services or transitional summaries. |
| **Zone services** | Zone PHC episodes store **membership snapshot** + optional **per-tree allocations**; do not infer history from current zone geometry alone. |
| **Jobs / quotes / requests** | Episodes optionally set **`jobId`** when work is scheduled from CRM. Job UI loads episodes via **query** on `jobId`. Invoice/job **line items** are **billing projections** (optional `episodeId` for traceability). Quote `serviceType: "treatment"` maps to structured PHC when user attaches an episode or template—not the other way around. |
| **Species catalog** | Link taxa to species; show risks on `view-tree`. |
| **Weather** | Snapshot on episode (wind, temp, precipitation) for pre-application context. |
| **Dashboard** | After structured data exists: upcoming episodes, open observations (Teams-heavy). |
| **Search** | Extend [search.md](./search.md) for taxa/products/programs. |
| **Notifications** | Later: REI, retreat windows. |
| **AI** | Assistive only: prompts from observations and org-entered product notes; **never** authoritative regulatory text. |

---

## 10. Compliance & risk

### 10.1 Compliance v1 (narrow slice) — ship this first

**Do not** build “compliance mode by org/region” as a **configuration framework** in v1—that is a multi-feature platform. **v1 compliance** means:

- **Required structured fields on application lines** when “documentation strict” flag is on (org-level boolean or simple preset): e.g. product name, rate, unit, applied date, applicator display name or ID field.
- **Immutable completion metadata** on episode: `completedAt`, `completedBy`, optional void/reversal record instead of silent edits.
- **Optional** license / applicator credential fields (stored, not validated against government APIs).
- **Customer-safe redaction** on shared/restricted views: hide or summarize chemical detail per rules in §10.2.

**Defer:** regional rule packs, automatic REI enforcement, notification workflows, complex conditional required fields.

### 10.2 Product + legal posture

- **Audit:** prefer append-only application lines with `voided` + reason rather than silent mutation.
- **Marketing:** “supports documentation” / “helps maintain records”—not “ensures compliance.”
- **Legal review** before any “compliance-ready” claim.

---

## 11. Product risks

| Risk | Mitigation |
|------|------------|
| **PHC too generic** | **Woody-first v1** (§1.4); defer turf/lawn-specific flows. |
| **Drawer / UI overload** | Primary entry at **tree, zone, job**; minimal PHC hub. |
| **CRM dictates domain model** | **PHC operational truth**; jobs/invoices **reference and summarize** (§5, §9). |
| **AI overreach** | Education + checklists only; no licensed-advice positioning (§1.3, §9). |
| **Zone history false precision** | **Snapshot + optional allocations** mandatory design rule (§6–§7). |
| **Duplicate events everywhere** | **Single source of truth** + timeline projection (§6 item 4). |
| **Overbuilding Phase 1** | Phase 1 success = *“I can log what I sprayed on this tree properly.”* Ship **observations + episodes + application lines + timeline** only; defer programs, zone PHC, reporting, TreeEvent dual-write, and CRM polish per §12. |

---

## 12. Phased roadmap (execution order)

This order **matches** “structured capture before reporting” and **biases** toward dedicated records + historical correctness.

### Phase 1 — Tree PHC core (**ruthlessly small**)

**Ship when:** a Pro user can complete: *“I logged what I sprayed on this tree properly.”*

**In scope**

- **Observation** CRUD + **treatment episode** CRUD + nested **application lines** on **trees** (woody).
- **`view-tree`** PHC timeline reading **only** from PHC subcollections (§6.1-B).
- **Canonical observation ↔ episode UX** (§8.2.1) and **Pro ad hoc** product fields on lines (§6.1-C).

**Explicitly out of scope for Phase 1**

- Org **product library**, job/invoice linkage, programs, zone PHC, exports/dashboards, `tree_events` dual-write, compliance “strict mode” beyond minimal required fields if any.

### Phase 2 — Product library & CRM linkage

- Org **product library** (Teams-tier primary).
- **`episode.jobId`** set from job workflow; job screen queries episodes; **invoice/line-item mappers** pull summaries from PHC (`lib/workflow/line-item-mappers.ts` pattern).

### Phase 3 — Zone PHC

- Zone-level episodes with **tree membership snapshot at treatment time** + optional **explicit per-tree allocation** records.
- Reconcile UX with existing `ZoneServiceRecord` (summary, migration path, or dual-write strategy—decide in implementation).

### Phase 4 — Reporting, dashboard, advanced compliance

- Property/zone exports (PDF/CSV), dashboard widgets, open observations / overdue retreats.
- **Then** expand compliance (regional packs, notifications, etc.) if still prioritized.

**Foundations (parallel to Phase 1 start):** `types/phc.ts`, Zod, Firestore rules, indexes, small **taxa seed** + import script (`scripts/`). No separate “Phase 0” gate that delays Phase 1 if ADR in §6 is already signed.

---

## 13. Analytics & success metrics

- Adoption: % of **Pro** vs **Teams** orgs with ≥1 PHC episode in 30 days (segment by tier).
- Time to log: tree open → saved episode + first line.
- Data quality: % lines with product + rate when strict documentation enabled.
- Retention: aggregate correlation with invoice volume (privacy-conscious).

---

## 14. Open questions (remaining)

1. **Taxon catalog:** global-only vs org fork from global?
2. **Inventory/stock:** out of scope until post–product library?
3. **Billing models:** per-tree vs per-zone vs flat—**presentation** on jobs; must not reshape PHC episode model.

**Resolved by default in v1.2 (reopen only if engineering forces):** PHC **`tree_events` dual-write** — **deferred past Phase 1** (§6.1-B).

---

## 15. Appendix — key file references

| Area | Path |
|------|------|
| Service & history types | `types/tree.ts` |
| Zone bulk services | `types/zone-service.ts`, `app/drawers/view-zone.tsx` |
| Jobs / line items | `types/job.ts`, `components/workflow/WorkflowLineItemsEditor.tsx` |
| Quotes / requests | `types/quote.ts`, `app/drawers/request-form.tsx` |
| Species care | `types/species.ts`, `app/drawers/restricted-tree/PublicCommunityContent.tsx` |
| History UI | `components/trees/add-history-entry-form.tsx`, `components/trees/tree-history-view.tsx` |
| Entry routes | `app/drawers/entry-form/index.tsx`, `components/trees/add-record-type-popover.tsx` |
| Firestore access | `lib/firebase/firestore.ts` |
| AI context | `lib/ai-chat/context-builder.ts`, `lib/ai-chat/suggestions.ts` |
| Drawers | `lib/drawers.ts`, `lib/i18n/navigation.ts` |

---

## 16. Implementation readiness

Before the first PHC PR merges:

- [ ] §**6** ADR table signed (all six rows) + §**6.1** locks (A–C).
- [ ] §**5** entity rules + **job↔episode direction** reviewed by tree history + jobs owners.
- [ ] §**8.2.1** UX flows agreed (product + design).
- [ ] Tier gates sketched against `SubscriptionTier` / drawer visibility.
- [ ] Zone attribution fields present in **first** zone PHC schema (even if UI ships later).
- [ ] Phase 1 scope matches §**12** “out of scope” list (no scope creep).

Optional follow-up: a short **go / no-go / fix-first** review that scores each Phase 1 story against these checkboxes.

---

*Document version: 1.2 — planning + execution prerequisites + execution locks; split implementation tickets by §12 phases.*
