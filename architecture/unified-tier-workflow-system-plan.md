# Plan: One system for all tiers (workflow + record keeping)

**Companion doc:** [tiers-workflow-vs-record-keeping-audit.md](./tiers-workflow-vs-record-keeping-audit.md) (current state and file index).

**Goal:** Replace parallel **Teams-only workflow** vs **Home/Plus/Pro record-keeping** implementations with a **single coherent product model**: one way to think about “work on a tree,” one capability matrix by tier, and one navigation/UX pattern—while preserving revenue differentiation through **limits and depth**, not duplicate UX paradigms.

---

## 1. Vision

- **Users** experience one mental model: **everything about a tree** (history, scheduled care, and business documents) lives in **one place**, with actions gated by **what their plan allows**, not by which subsystem they landed in.
- **Engineering** maintains **one authorization/capability layer**, **one primary navigation contract** (`lib/drawers.ts` + guards), and **clear extension points** for tier upsell instead of `useTeamsOnlyAccess` sprinkled across 20+ files.

---

## 2. Guiding principles

| Principle | Implication |
|-----------|-------------|
| **Capabilities, not hooks** | Replace repeated `useTeamsOnlyAccess` / raw `isTierInGroup` with a **`useAppCapabilities()`** (or similar) derived from `userDoc` + org once per route subtree. |
| **Same objects, tiered depth** | Prefer **one** set of entities (e.g. job-like work, client links) with **fields or scopes** that unlock with tier—not two unrelated data paths. |
| **Progressive disclosure** | Lower tiers see **subset** of columns, **fewer** list rows, or **personal** scope; upgrade explains **what unlocks**, not “wrong app.” |
| **Nav truth = runtime truth** | `visibleForTiers` in `lib/drawers.ts` must match **Firestore rules** and **work-area** guards; document a single matrix. |
| **Tree is the hub** | View tree footer/tabs should not **swap paradigms** (workflow vs record); use **one primary action row** with overflow for tier-specific actions. |

---

## 3. Target architecture (high level)

### 3.1 Capability matrix (product + engineering)

Define a **versioned** contract (TypeScript) e.g. `AppCapabilities`:

- **Record keeping:** create/edit tree history entries (`entry-form`), view `TreeHistoryView`.
- **Clients & properties:** list/create/edit clients & properties (today Pro/Teams; Plus partial).
- **Workflow pipeline:** create/view requests, quotes, jobs, invoices **scoped to org** (today Teams only).
- **Team collaboration:** shared inbox, team workflow settings, assignees (Teams-heavy).
- **Map / dashboard:** which filters (e.g. jobs on map) apply.

Each capability is **`boolean | 'limited'`** where needed, with optional numeric limits (max open jobs, etc.) if product wants.

**Deliverable:** `lib/capabilities/*` (or `hooks/useAppCapabilities.ts`) + unit tests from **fixture userDocs** per tier.

### 3.2 Replace ad-hoc gates

| Current | Target |
|---------|--------|
| `useTeamsOnlyAccess` everywhere | `capabilities.workflowPipeline` (name TBD) |
| `useProOrTeamsAccess` + inline `isTierInGroup` | `capabilities.clientsProperties`, `capabilities.zones`, etc. |
| `WorkflowActiveWorkCard` returns null non-Teams | Card **renders** with **empty + upsell** or **simplified “upcoming”** for lower tiers (product decision) |

### 3.3 Data model direction

**Decision fork (must resolve in Phase 0):**

- **Option A — Same collections, scoped by tier**  
  Home/Plus users write to **same** `requests`/`jobs`/… collections with `scope: 'personal' | 'org'` and rules enforce **read/write** by membership + tier.  
  *Pros:* one reporting pipeline. *Cons:* rules + migration complexity.

- **Option B — Lightweight entities for lower tiers**  
  Lower tiers use **tree history + “simple job”** blobs; **import/promote** to full workflow when upgrading.  
  *Pros:* simpler rules early. *Cons:* two paths until merge.

The plan should **pick one** before large engineering. This doc recommends **Option A** long-term if the business wants **one** operational truth; use **feature flags** to roll out.

### 3.4 Navigation (`lib/drawers.ts`)

- Build **`DRAWER_TIER_MATRIX.md`** (or section in this file) listing **drawer id → minimum capability** instead of only `visibleForTiers` arrays.
- Optionally generate `visibleForTiers` from the matrix to avoid drift.
- Keep **`work-area-content.tsx`** `canUserAccessDrawer` aligned with the same matrix.

### 3.5 View tree — single shell

- **Footer:** One **primary** CTA: “**Work on this tree**” or split **Record** / **Work order** as **equal** secondary actions—not mutually exclusive modes by tier string.
- **Operations tab:** Merge **service summary**, **tags**, and **pipeline snapshot** into one scroll with consistent cards; avoid Teams-only card as the only “professional” block.
- **Remove dead props:** e.g. `TreeActivityTab` `hasTeamsAccess` if still unused.

---

## 4. Phased roadmap

### Phase 0 — Decisions & matrix (1–2 weeks, product + tech lead)

- [ ] Choose data model fork (A vs B) and **upgrade story** (promote vs net-new).
- [ ] Approve **capability matrix** v1 (what Home/Plus/Pro/Teams can do).
- [ ] Approve **UX** for non-Teams on workflow surfaces (hide vs read-only vs upsell).
- [ ] Add **`DRAWER_TIER_MATRIX`** draft next to this plan.

### Phase 1 — Capabilities foundation (engineering)

- [ ] Implement `getAppCapabilities(userDoc, orgCtx)` in `lib/capabilities/` with tests.
- [ ] Add **`useAppCapabilities()`** hook (memoized).
- [ ] Migrate **one** vertical slice end-to-end (e.g. `WorkflowActiveWorkCard` + `view-tree` footer) behind a **feature flag** `unifiedCapabilities`.

### Phase 2 — Replace workflow gates (engineering)

- [ ] Replace `useTeamsOnlyAccess` in: workflow list/detail drawers, forms, `WorkflowActiveWorkCard`, dashboard map sections—**using capabilities**.
- [ ] Consolidate **upgrade** copy to a **small set** of i18n keys fed by capability id (avoid 20 similar strings).
- [ ] Update **`lib/drawers.ts`** visibility to match matrix (or generate from capabilities at runtime—only if perf OK).

### Phase 3 — Record keeping + workflow UX (product + engineering)

- [ ] Redesign **view-tree** tabs/footer per §3.5; user testing on Home vs Teams accounts.
- [ ] Align **`entry-form`** and workflow forms: shared **dirty** patterns already in `WORKFLOW_DRAWERS`; rename set if “workflow” is misleading.

### Phase 4 — Backend & security

- [ ] Update **`firebase/firestore.rules`** to enforce the same capability model (org vs personal, tier not in rules if impossible—then **claims** or **Cloud Functions**).
- [ ] Audit **Stripe** webhooks (`lib/stripe-webhook-handlers.ts`) so tier changes **invalidate** capability cache client-side.

### Phase 5 — Cleanup & deprecation

- [ ] Remove `useTeamsOnlyAccess` if fully subsumed (or keep thin wrapper for one release).
- [ ] Delete duplicate tier checks in `map-toolbar`, `add-tree-form`, etc.
- [ ] Update **FAQ** and internal docs (`current-workflow-audit.md`, etc.).

---

## 5. Feature flags & rollout

| Flag | Purpose |
|------|---------|
| `unifiedCapabilities` | Serve capabilities from new module; fallback to old hooks if false. |
| `workflowLowerTiers` (example) | Show read-only pipeline or upsell for Home/Plus—product-dependent. |

Roll out: **internal → beta → % rollout → default on**; keep old hooks for **one release** after parity.

---

## 6. Testing strategy

- **Unit:** `getAppCapabilities` for fixture users (Home, Plus, Pro, each Team size, expired subscription → Home).
- **Integration:** Open each **tier-restricted drawer** via URL; assert **redirect/block** matches matrix (`work-area-content`).
- **E2E (optional):** View tree Operations + footer for two tiers; snapshot critical CTAs.

---

## 7. Success metrics

- **Support:** Fewer “I can’t find requests” tickets from Pro users (if Pro gains access) or clearer upsell clicks.
- **Engineering:** Count of files importing `useTeamsOnlyAccess` → **0** (or 1 adapter).
- **Product:** Single **help** article describing “how work on trees works” across tiers.

---

## 8. Risks & mitigations

| Risk | Mitigation |
|------|------------|
| Scope explosion | Time-box Phase 0; ship **capabilities** without changing all tiers first. |
| Security regression | Rules review + staged rollout; no broad Firestore open reads. |
| Revenue cannibalization | Matrix owned by product; **limits** on lower tiers instead of hiding entire value. |
| Performance | Capabilities computed once per session or memoized with `userDoc` subscription hash. |

---

## 9. Open questions (checklist)

1. Should **Plus** see **Clients** drawer or only properties-without-clients forever?
2. Are **invoices** ever allowed below Teams (unlikely) or always upsell?
3. Is **map Jobs filter** the same entity as **workflow jobs** for all tiers?
4. Do we need **per-org** overrides (enterprise) on top of tier?

---

## 10. References

- [tiers-workflow-vs-record-keeping-audit.md](./tiers-workflow-vs-record-keeping-audit.md)
- `lib/drawers.ts`, `lib/utils/tier-utils.ts`, `stores/navigation-store.ts` (`WORKFLOW_DRAWERS`)
- `components/work-area/work-area-content.tsx` (`canUserAccessDrawer`)

---

*Version 1.0 — living document; bump version when Phase 0 decisions are recorded.*
