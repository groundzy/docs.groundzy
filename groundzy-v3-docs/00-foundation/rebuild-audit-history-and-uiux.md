# Groundzy v3 Rebuild Audit: History Systems and UI/UX Consistency

## Executive Summary

**History and activity**

The repository implements **multiple overlapping concepts** for “what happened” on a tree, on the platform, and in commercial workflow:

1. **Tree history** — Embedded `TreeHistory` on the tree document (`serviceHistory`, `inspectionHistory`, etc.) plus a **Phase 4** append-only subcollection `tree_events/{treeId}/events/{eventId}` with dual-read/dual-write in `lib/firebase/firestore.ts`.
2. **Work items** — A separate Firestore collection `work_items` that aggregates operational and mirrored workflow rows, with **dual-write** from tree history entries and from CRM workflow documents (`types/work-item.ts` marks `workflow_*` kinds as **LEGACY_MIRROR**).
3. **Merged “Activity” UI** — `TreeHistoryView` can combine **legacy tree history** with **`work_items`** rows when a **user preference** enables it (`workItemsActivityTimelineEnabled`); this is **not** expressed as a simple Home-vs-Teams tier check in code (see §1).
4. **Public / community summaries** — `PublicActivityEntry` and `publicActivity` / `activity` on public-facing types (`types/tree.ts`) are a **redacted, display-oriented** activity shape for unrestricted views, distinct from full `HistoryEntry` records.
5. **Weather** — “Timeline” here means **external forecast API** (`lib/weather/timeline-client.ts`, `/api/weather/timeline`), not user activity history.

**UI/UX**

The app has a **real design system** (`components/ui/`, `components/drawer-layout/`, documented **DrawerShell** rules), but **many documented exceptions** and **per-drawer visual overrides** (`docs/audits/drawer-visual-inventory.md`). Internal audits score **maintainability below architecture** and call out **megafiles** and **registry/ordering drift** (`docs/audits/drawer-system-current-state.md`). Several drawers intentionally diverge (search without `DrawerShell`, full-bleed wizards, large forms), which preserves flexibility but works against a single “one app” feel.

---

## Why This Audit Matters

For **Groundzy v3**, shipping a **unified history model** and **consistent shell/interaction patterns** is constrained by today’s **parallel storage paths** (tree doc + `tree_events` + `work_items` + workflow docs + public summaries) and **UI exception tables**. Without addressing these, a rebuild risks **re-encoding the same splits** or **breaking** dual-write and merge behavior that production data may depend on. This audit maps **what exists** so v3 can replace it deliberately rather than accidentally.

---

## 1. History System Findings

### 1.1 Tree history (embedded + events)

| Field | Detail |
|-------|--------|
| **Name / concept** | `TreeHistory` — arrays of typed records (`ServiceRecord`, `InspectionRecord`, `NoteRecord`, `MeasurementRecord`, `AccessRecord`); union `HistoryEntry`. |
| **Purpose** | Chronological **tree-centric** log of care actions and notes. |
| **Files** | `types/tree.ts`; CRUD in `lib/firebase/firestore.ts` (`addTreeHistoryEntry`, `updateTreeHistoryEntry`, `deleteTreeHistoryEntry`, etc.); UI in `components/trees/tree-history-view.tsx`, `components/trees/add-history-entry-form.tsx`, `app/drawers/entry-form/`; large integration in `components/trees/add-tree-form.tsx`. |
| **Tiers / context** | Core tree product (all tiers that have trees). Not Teams-specific. |
| **Storage** | **Dual:** embedded `tree.history` **and** subcollection `tree_events/{treeId}/events` (“Phase 4”; comments say embedded replaced for scale / migration path). Reads prefer `tree_events` when present (`listTreeEvents`, `treeEventsToHistory`). |
| **UI** | **Activity** tab on `view-tree` (`app/drawers/view-tree/components/TreeActivityTab.tsx`) passes `tree.history` into `TreeHistoryView`. |
| **Overlap** | Same logical entries may exist in **embedded history**, **`tree_events`**, and (if org-scoped) **`work_items`** (see §1.3). |

### 1.2 Work items (`work_items` collection)

| Field | Detail |
|-------|--------|
| **Name / concept** | `WorkItem` — “one actionable unit” of work; kinds include tree-native types **and** `workflow_request`, `workflow_quote`, `workflow_job`, `workflow_invoice` (**legacy mirrors** of CRM docs). |
| **Purpose** | **Cross-entity index** for planning, scheduling, dashboards, and merged timelines; architecture doc positions it as the long-term **system of record** for ops while tree history stays a readable log (`docs/architecture/work-item-system-architecture.md`). |
| **Files** | `types/work-item.ts`; `lib/firebase/work-items.ts`; `lib/workflow/work-item-adapters.ts` (mapping from `HistoryEntry`, `Request`, `Quote`, `Job`, `Invoice`, zone services); hooks `hooks/useWorkItems.ts`; consumers include `TreeActivityTab`, `ViewZoneWorkItemsIndexSection`, `dashboard` (upcoming merge), `view-invoice` (linked work items), `profile` preview. |
| **Tiers / context** | Data is **`organizationId`-scoped** queries. `syncWorkItemFromTreeHistoryEntry` **returns early if `tree.organizationId` is missing** (`lib/firebase/work-items.ts`). Mirrors fire for workflow entities when dual-write is enabled. **Not** labeled “Teams only” in these functions—**org context** matters. |
| **Storage** | Top-level collection `work_items` (see `COLLECTION` constant in `work-items.ts`). |
| **UI** | Rows rendered inside `TreeHistoryView` when `workItems` prop is set; zone section lists work items; dashboard can merge upcoming with work items when flag enabled. |
| **Overlap** | **Dual-write** from tree history and workflow saves (`fireAndForgetSyncTreeHistory`, `fireAndForgetSyncRequest`, etc.) controlled by `isWorkItemsDualWriteEnabled` (`lib/workflow/work-item-flags.ts`, default on unless user pref disables). **Fragmentation:** same business event can be represented in **CRM collections**, **`work_items` mirror row**, and **tree `HistoryEntry`**, with dedupe in UI via `source.historyEntryId` (`components/trees/tree-history-view.tsx`). |

### 1.3 Merged Activity timeline (tree)

| Field | Detail |
|-------|--------|
| **Name / concept** | “Merged timeline” / “Work timeline (preview)” — copy in `lib/i18n/messages.ts` and `ProfileWorkItemsCollapsible.tsx`. |
| **Purpose** | Single chronological stream: **creation row + tree history entries + work items** (excluding work items that duplicate a history id). |
| **Files** | `TreeHistoryView` (`displayEntries` combining `creation`, `filteredEntries`, `workItemRows`); `TreeActivityTab.tsx` loads `useWorkItemsForTree` only when `isWorkItemsActivityTimelineEnabled(prefs)` is true. |
| **Tiers / contexts** | Gated by **`userDoc.preferences.workItemsActivityTimelineEnabled === true`** (`lib/workflow/work-item-flags.ts`). Default **false** (“until rollout”). **`hasTeamsAccess` is passed into `TreeActivityTab` but unused** (`_hasTeamsAccess`) — tier is **not** the switch in this file. |
| **Storage** | No separate store; merges in memory from `tree.history` + `work_items` query. |
| **UI** | Activity tab on view-tree; profile collapsible describes the preview. |
| **Overlap** | Addresses **duplication** between history arrays and `work_items` only at **render** time; underlying data duplication remains unless dual-write is off. |

### 1.4 Workflow CRM documents (requests, quotes, jobs, invoices)

| Field | Detail |
|-------|--------|
| **Name / concept** | First-class entities with **status enums** (e.g. `Request.status`, `Quote.status`, `Job.status`, `Invoice` states) — see `types/request.ts`, `types/quote.ts`, `types/job.ts`, `types/invoice.ts`. |
| **Purpose** | Commercial pipeline; **not** implemented (in the types reviewed) as a separate append-only **status change log** collection—state lives on the **document**. |
| **Files** | `app/drawers/view-*`, `app/drawers/*-form`, `lib/workflow/*`, `docs/features/current-workflow-audit.md` (gaps in conversion/backlinks documented separately). |
| **Tiers** | **Teams** drawers for full workflow per `lib/drawers.ts` / `docs/audits/drawer-audit.md`. |
| **Mirrors** | `work_items` rows with `workflow_*` kinds and `source.workflowDocumentId` / `workflowCollection` (`types/work-item.ts`). |
| **Overlap** | Parallel representation of “what’s happening” in **CRM doc** vs **`work_items` mirror** vs **tree history** when jobs/services are recorded on trees. |

### 1.5 Public / community activity

| Field | Detail |
|-------|--------|
| **Name / concept** | `PublicActivityEntry`, `TreePublicSummary.publicActivity`, `Tree.activity` for public trees (`types/tree.ts`). |
| **Purpose** | **Privacy-safe, coarse** labels (e.g. year/season, no pricing) for public discovery — not a full audit trail. |
| **Storage** | `tree_public_summaries` mentioned in comments on `TreePublicSummary`. |
| **UI** | Restricted/public tree flows (`app/drawers/restricted-tree/`). |
| **Overlap** | Conceptually parallels “history” but **different shape and trust model** than `HistoryEntry`. |

### 1.6 Weather timeline (external)

| Field | Detail |
|-------|--------|
| **Name** | Weather API “timeline” (`lib/weather/timeline-client.ts`, `app/api/weather/timeline/route.ts`). |
| **Purpose** | Forecast hours for impact UI — **unrelated** to user activity history. |

### 1.7 Plant Health Care (PHC) — planned unified timeline language

| Field | Detail |
|-------|--------|
| **Docs** | `docs/features/plant-health-care.md` describes a **“Health care” timeline** (observations, episodes, jobs) and explicitly warns against **fragmented drawers** for PHC UX. |
| **Status** | Product/planning document; not treated here as fully implemented production parity with tree history. |

---

## 2. Evidence of Multiple History Systems

**Yes — the codebase contains several distinct “history-like” systems with overlapping responsibilities:**

1. **Tree-native history** (`TreeHistory` / `tree_events`) vs **`work_items`** vs **workflow CRM documents** — described explicitly in `docs/architecture/work-item-system-architecture.md` (tree history as **output log**, work items as **execution / planning** layer, workflow docs as **containers**).

2. **Code-level duplication path** — `fireAndForgetSyncTreeHistory` and related functions write **both** tree history (and events) **and** `work_items` when enabled (`lib/firebase/work-items.ts` + `work-item-flags.ts`).

3. **UI merge rather than single source** — `TreeHistoryView` dedupes **work items** that mirror a history id (`source.historyEntryId` vs `historyEntryIds` set) but **coexistence** of three layers (embedded, events, work_items) is inherent.

4. **Workflow status** — Modeled as **fields on entities**, not a unified platform-wide **event log** (no single `audit_events` collection was identified in this audit’s searches).

5. **Public activity** — Third representation for the same tree narrative for **public** audiences.

**Why this is a problem for product cohesion**

- Users can see **different** Activity content depending on **flags** and **whether** `work_items` loaded, not only on role.
- Engineers must maintain **adapters** (`work-item-adapters.ts`) and **dual-write** semantics indefinitely unless v3 collapses the model.
- **Tier vs org** is easy to misstate: work item sync keys off **`organizationId`**, not sidebar “Teams” alone.

---

## 3. Recommendation for a Unified History Model

*Conceptual only — grounded in repo evidence, not an implementation spec.*

1. **Single append-only event stream (conceptual)** for “things that happened” that **references** domain entities (tree, job, invoice) instead of **re-storing** full payloads in three places — aligns with the pain of dual-write and merge (`TreeHistoryView` + `work_items`).

2. **Clear separation of concerns** as already argued internally: **chronological narrative** (what the landowner reads) vs **operational work record** (scheduling, assignment, money) vs **commercial documents** — the existing architecture doc is a useful north star (`work-item-system-architecture.md`).

3. **Deprecate `workflow_*` mirror kinds** in favor of **either** canonical CRM **or** work items, not both long-term — `isLegacyMirrorWorkItemKind` in `types/work-item.ts` already signals **legacy**.

4. **One Activity UI contract** — Whether v3 keeps one Firestore collection or many, the **product** should expose **one** merged timeline rule set (no silent pref defaults hiding workflow rows).

5. **Public summaries** — Continue as a **derived, privacy-filtered projection** of the unified model, not a fourth independent author.

---

## 4. UI/UX Consistency Findings

| Finding | Evidence |
|---------|----------|
| **Binary DrawerShell rule + many exceptions** | `docs/audits/drawer-shell-classification.md` — `search`, `tree-add`, `edit-tree`, `multiple-add`, `ai-identifying-wand`, `groundzy-wizard` use **non-standard** root patterns by design. |
| **Per-drawer visual divergence** | `docs/audits/drawer-visual-inventory.md` — e.g. weather gradient, trees/clients `bg-transparent`, Pro forms `brandedDeepTealCardClass`, search **no DrawerShell**. |
| **Drawer count and registry drift** | `docs/audits/drawer-audit.md` — **56** drawer IDs; `docs/audits/drawer-system-current-state.md` — registry header undercounts, **four drawers share `order: 1.1`**. |
| **Megafiles / maintainability** | Same doc §9 — **~20** top-level files **>400 lines**; maintainability score **6/10**; Phase C extraction backlog (`view-zone`, `request-form`, `contact-us`, `ai-chat`, `profile`). |
| **Special-case wiring** | `docs/audits/drawer-audit.md` — **`withRetry`** only on **`entry-form`** chunk loads; inconsistent with other lazy imports. |
| **Comment / metadata mismatch** | `docs/audits/drawer-audit.md` — `ai-identifying-wand`: `showHeaderNav` vs comment “no back/forward” **disagree** — QA risk. |
| **Workflow dirty vs forms** | `WORKFLOW_DRAWERS` includes CRM workflow forms **and** `entry-form` for tree history (`stores/navigation-store.ts`, `docs/audits/drawer-system-current-state.md`); other form-like drawers may **not** get discard protection — intentional but uneven. |
| **Huge composite forms** | `add-tree-form.tsx` and similar — multi-thousand-line orchestration increases **one-off** UX variance. |

---

## 5. Most Fragmented Areas

1. **View tree** — Tabs (overview, activity, ops, ecology, media, system, optional community) + `TreeHistoryView` merge logic + optional work items — high **cognitive and code** surface (`app/drawers/view-tree/`).
2. **Search drawer** — **Documented exception** without `DrawerShell** (`docs/audits/drawer-shell-classification.md`) — structurally isolated from the default shell stack.
3. **AI flows** — `ai-identifying-wand`, `groundzy-wizard`, `ai-chat` — full-bleed / custom chrome vs standard detail drawers.
4. **Contact / inbox** — `contact-us` combines support inbox, threads, share-tree, access requests (`app/drawers/contact-us.tsx`) — multi-mode “mini app” inside one drawer id.
5. **CRM workflow** — List + view + form drawers per stage; card-list consistency documented, but **workflow audit** lists conversion/status gaps (`docs/features/current-workflow-audit.md`).

---

## 6. Design System / Shared Component Observations

| Topic | Assessment |
|-------|------------|
| **Shared system exists** | **Yes** — `components/ui/` (shadcn/Radix), `components/drawer-layout/` (`DrawerShell`, `DrawerScrollArea`, `DrawerFooter`, `ListContainer`, etc.). |
| **Partial standardization** | **Yes** — `docs/audits/drawer-visual-inventory.md` and **workflow step** color tokens (see `.cursor/rules/workflow-step-colors.mdc`, `app/globals.css`) show deliberate reuse in places. |
| **Bypassing shared patterns** | **Documented exceptions** are **explicit** (search, wizards, tree-add hosts); **not** necessarily accidental, but they **multiply** interaction models. |
| **Fragmented custom UI** | Large drawer-specific components, **one-off** scroll regions (chat/contact threads per `docs/audits/drawer-shell-classification.md`), and **branded** card classes for Pro forms create **visual families** that read as sub-products. |

---

## 7. Groundzy v3 Recommendations

**History / tracking**

- Converge on **one conceptual model** for user-visible timelines (tree + org + workflow), with **explicit** rules for projections (public, dashboard, PHC).
- Plan **migration** off **dual-write** and **`LEGACY_MIRROR`** workflow kinds once a single source of truth exists.
- Treat **weather timeline** and **user activity** as **different domains** in naming (avoid “timeline” overload in UX copy).

**UI / UX / architecture**

- Enforce a **small set of drawer archetypes** (list, detail, form, tool, chat) with **fewer** root exceptions, or **compose** exceptions from shared primitives.
- Reduce **megafiles** as a **v3 structural goal** (evidence: line budgets and `verify:drawers` warnings in docs).
- Align **feature flags** (e.g. Activity merge) with **product tiers** or **clear global defaults** so behavior is predictable.
- Resolve **registry hygiene** (ordering, drawer count comments) via automation or codegen in v3.

---

## 8. File Reference Appendix

### History / timeline / activity

- `types/tree.ts` — `TreeHistory`, `HistoryEntry`, `TreeEvent`, `PublicActivityEntry`, public summary activity fields.
- `lib/firebase/firestore.ts` — `tree_events`, `addTreeHistoryEntry`, `listTreeEvents`, `treeEventsToHistory`.
- `components/trees/tree-history-view.tsx` — merge of history + `workItems` + creation row.
- `app/drawers/view-tree/components/TreeActivityTab.tsx` — prefs gate for work items feed.
- `lib/firebase/work-items.ts` — `work_items` CRUD, dual-write helpers, org-scoped queries.
- `lib/workflow/work-item-adapters.ts` — mappings from history and workflow entities.
- `lib/workflow/work-item-flags.ts` — `workItemsDualWriteEnabled`, `workItemsActivityTimelineEnabled`, `workItemsDashboardUpcomingEnabled`.
- `hooks/useWorkItems.ts` — React Query hooks for work items.
- `docs/architecture/work-item-system-architecture.md` — intended roles of history vs work items vs workflow.

### Workflow / CRM

- `types/work-item.ts` — `WorkItem`, `isLegacyMirrorWorkItemKind`, `WorkItemSource`.
- `types/request.ts`, `types/quote.ts`, `types/job.ts`, `types/invoice.ts` — status models.
- `docs/features/current-workflow-audit.md` — gaps, inconsistencies, technical debt.
- `lib/workflow/*.ts` — conversion, line items, adapters.

### Weather (not user history)

- `lib/weather/timeline-client.ts`, `app/api/weather/timeline/route.ts`.

### Drawer / UI / audits

- `lib/drawers.ts`, `lib/drawer-registry.ts` — registration and metadata.
- `docs/audits/drawer-shell-classification.md` — shell vs exceptions.
- `docs/audits/drawer-visual-inventory.md` — per-drawer UI patterns.
- `docs/audits/drawer-audit.md` — full inventory, wiring notes.
- `docs/audits/drawer-system-current-state.md` — megafiles, scores, debt.
- `components/drawer-layout/` — shared drawer primitives.
- `stores/navigation-store.ts` — `WORKFLOW_DRAWERS`, drawer history.

### PHC (planning)

- `docs/features/plant-health-care.md` — timeline / fragmentation concerns.

---

## Ambiguity / not verified in this pass

- **Exact** Firestore collections for every notification or inbox message type (dashboard combines unread hooks — details not fully traced).
- Whether **all** production users have `organizationId` on trees for dual-write — **inferred** from `syncWorkItemFromTreeHistoryEntry` guard.
- Full **PHC** implementation status vs design doc.
- **Per-entity status change history** — if any subcollection stores audit trails for requests/jobs, it was not surfaced in the `types/` grep sample; default assumption: **state on document** unless extended elsewhere.

---

*Audit performed against repository structure and docs; cite paths when extending this document.*
