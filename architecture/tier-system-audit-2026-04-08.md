# Groundzy tier system — full repository audit (April 2026)

**Scope:** `app.groundzy` only (this repo).  
**Goal:** inventory **canonical tier strings**, where they are **derived**, how **access is enforced** (UI vs API vs server policy vs Firestore rules), and **known inconsistencies / risks**.

**Related existing docs (still useful):**

- `docs/audits/pricing-tier-subscription-audit-2026-04.md` — broader Stripe + marketing cross-app notes (includes `auth.groundzy` references).
- `docs/architecture/tiers-workflow-vs-record-keeping-audit.md` — workflow vs record-keeping split + file index (partially stale vs current `lib/drawers.ts`).
- `docs/architecture/unified-tier-workflow-system-plan.md` — intended consolidation roadmap.
- `docs/operations/backfill-subscription-tier.md` — operational backfill for legacy Plus tier integrity issues.

---

## 1) Canonical tier model (code)

### 1.1 `SubscriptionTier` union (UI + navigation)

The canonical tier union for drawers/navigation is:

- `Home`
- `Plus`
- `Pro`
- `Small Team`
- `Mid Team`
- `Large Team`
- `Enterprise`

**Source:** `lib/drawer-registry.ts` (`export type SubscriptionTier = ...`).

### 1.2 Signup / checkout “plan keys” (lowercase)

`types/signup-flow.ts` defines marketing/checkout-oriented keys:

- `PlanTier = 'home' | 'pro' | 'teams'`
- `TeamSize = '10' | '25' | '50' | 'enterprise'`

`PRICING_TIERS` in the same file is **display/marketing copy** (not enforcement).

### 1.3 Ambiguous legacy string: `Teams`

Multiple flows still write or accept `subscription.tier: "Teams"` (capital T) or metadata tier `"Teams"` / `"teams"`.

**Runtime normalization:** `normalizeTier()` maps `teams` / `Teams` → `Small Team` by default.

**Source:** `lib/utils/tier-utils.ts`.

**Implication:** `"Teams"` is not a `SubscriptionTier` drawer tier; it is a **checkout/onboarding label** that should collapse to a concrete team size tier via Stripe price mapping + webhook updates.

---

## 2) Effective access vs “raw tier” (billing state)

### 2.1 Effective tier used across the app

`getEffectiveSubscriptionTier(userDoc)` returns:

- `Home` always as `Home` (even if subscription fields are weird)
- `Plus` only if subscription status is in `active` or `trialing`
- `Pro` + all team tiers only if subscription status is `active` or `trialing`
- otherwise **downgrades effective access to `Home`** (even if `subscription.tier` still says `Pro`, `Plus`, etc.)

**Source:** `lib/utils/tier-utils.ts`.

**Important nuance:** `Plus` is treated as a **paid tier** gated by subscription status the same way as `Pro`/teams.

### 2.2 What “paid tier incomplete” means

`hasPaidTierIncomplete` / `hasActivePaidSubscription` compare **raw** `subscription.tier` from the user doc to subscription status.

**Source:** `lib/utils/tier-utils.ts`.

This is used for UX flows like “finish payment”, but feature gating should generally use `getEffectiveSubscriptionTier` + capabilities.

---

## 3) Product capabilities matrix (implemented)

### 3.1 Client-side capabilities (`useCapabilities`)

`lib/capabilities.ts` defines `CapabilityKey` and `getCapabilitiesForTier(tier)`.

**Implemented rules (effective tier):**

- **Home:** `workitems.read_tree`, `workitems.write_tree`
- **Plus:** adds `legacy.pro_plus_teams_surfaces` + `crm.properties`
- **Pro:** adds `crm.clients` (and keeps the legacy bundle + properties)
- **Teams (any size):** adds `workflow.pipeline`, `workitems.read_org`, `map.layers_jobs`

**Hook:** `hooks/useCapabilities.ts` memoizes capabilities from `getEffectiveSubscriptionTier`.

### 3.2 Server entitlements mirror (`users/{uid}.entitlements`)

`lib/groundzy/entitlements/resolve.ts` computes `UserEntitlements` using the same broad rules as `lib/capabilities.ts`:

- `workflowPipeline` → team tiers only
- `crmProperties` → Plus + Pro + teams
- `crmClients` → Pro + teams
- `workItemsReadOrg` / `mapLayersJobs` → teams only

**Schema:** `lib/groundzy/entitlements/types.ts`.

**Sync:** Stripe webhook handlers call `syncEntitlementsForUser` after subscription mutations (`lib/stripe-webhook-handlers.ts`).

---

## 4) Usage limits (numeric)

### 4.1 Trees / AI / PlantNet (Identifying Wand)

**Source of truth table:** `lib/ai-usage.ts`

- **Trees:** Home 5, Plus 50, Pro+ unlimited (`-1`)
- **AI chat + tree summary:** Home 20, Plus 50, Pro+ unlimited (`UNLIMITED_USAGE = -1`)
- **PlantNet / Wand:** Home 10, Plus 50, Pro+ unlimited

**Note:** `types/signup-flow.ts` marketing claims “unlimited” for Pro/Teams, but the app’s implemented limits are **unlimited** for Pro/Teams in `lib/ai-usage.ts` (not capped at 500 anymore). If any external doc still mentions numeric caps, treat it as stale unless sourced from current `lib/ai-usage.ts`.

### 4.2 Period logic

Usage keys roll on a **signup-day anchored month** when `createdAt` exists; otherwise calendar month.

**Source:** `getCurrentPeriodKey` in `lib/ai-usage.ts`.

### 4.3 Enforcement surfaces (API routes)

These routes compute tier via `getEffectiveSubscriptionTier` and enforce limits:

- `GET /api/trees/usage` — `app/api/trees/usage/route.ts` (includes zone aggregate counts toward tree limit)
- `GET /api/ai/usage` — `app/api/ai/usage/route.ts`
- `POST /api/ai/chat` — `app/api/ai/chat/route.ts`
- `POST /api/ai/tree-summary` — `app/api/ai/tree-summary/route.ts` (counts against same `aiUsage` counter as chat)
- `POST /api/species/identify/reserve` — `app/api/species/identify/reserve/route.ts`

---

## 5) Navigation + drawer gating (`visibleForTiers`)

### 5.1 Registry

`lib/drawers.ts` registers drawers and assigns `visibleForTiers`, `category`, `role`, etc.

**Key tier groups used in this file:**

- `HOME_PLUS_PRO_AND_TEAMS` includes Home/Plus/Pro/all team tiers.
- `TEAMS_ONLY_TIERS` includes all team tiers.

### 5.2 CRM surfaces (clients/properties)

`clients-properties` / `properties` / `clients` are visible for:

- `Plus`, `Pro`, and all team tiers

**Source:** `lib/drawers.ts` (`registerDrawer("clients-properties", ...)` etc.).

**Mismatch note:** older docs claimed Pro-only CRM navigation; current registry includes **Plus**.

### 5.3 Workflow pipeline drawers (org workflow)

These are generally **Teams-only** in navigation:

- `requests`, `add-request`, `edit-request`
- `quotes`, `add-quote`, `edit-quote`
- `jobs`, `add-job`, `edit-job`
- `invoices`, `add-invoice`, `edit-invoice`

**Source:** `lib/drawers.ts` (`visibleForTiers: [...TEAMS_ONLY_TIERS]`).

### 5.4 Workflow *detail* drawers (important inconsistency)

These drawers are registered with `visibleForTiers: HOME_PLUS_PRO_AND_TEAMS` (i.e., **not Teams-only**):

- `view-request`
- `view-quote`
- `view-job`
- `view-invoice`

**Source:** `lib/drawers.ts`.

**Why this matters:**

- Navigation metadata allows **non-Teams** users to open these drawers (and deep links won’t be blocked by `canUserAccessDrawer` in `components/work-area/work-area-content.tsx`).
- Several of these drawer components still call `useTeamsOnlyAccess()` and **early-return an upgrade screen** for non-Teams users, even if the user is a **v3 participant** with legitimate access via `participantPrincipalIds`.

This is the largest **tier/product UX inconsistency** found in this audit.

### 5.5 Participant Work (v3)

`participant-work` is registered for **all** `HOME_PLUS_PRO_AND_TEAMS` tiers.

**Source:** `lib/drawers.ts`.

The list API is **not tier-gated**; it is **participant-gated** by Firestore fields + Admin SDK reads:

- `GET /api/workflow/participant-list` — `app/api/workflow/participant-list/route.ts`
- Server list implementation — `lib/server/participant-workflow-list.ts`

**Risk:** combined with §5.4 + `view-request` gating, a Home/Plus user who is a participant can see items in **Work**, but may be blocked from viewing details in `view-request` / `view-quote` / etc.

---

## 6) URL guard behavior (deep links)

`components/work-area/work-area-content.tsx`:

- Computes `userTier` via `getEffectiveSubscriptionTier`
- Blocks opening a drawer if `visibleForTiers` excludes that tier

**Important consequence:** because `view-request` includes non-Teams tiers in `visibleForTiers`, the work-area guard **will not** protect Teams-only workflows from deep links; the drawer-level `useTeamsOnlyAccess()` logic is doing the real gating (and is currently incompatible with participant access).

---

## 7) Stripe mapping (app)

### 7.1 Price IDs

**Source:** `lib/stripe-config.ts`

- Maps checkout tiers + billing cycle + team size → Stripe **price id**
- Maps Stripe price id → tier string via `getTierFromPriceId` (webhook fallback when metadata is stale)
- Maps `enterprise` team size → **Large Team** prices (billing equivalence)

### 7.2 Checkout session creation

**Route:** `app/api/stripe/create-checkout-session/route.ts`

- Uses `getPriceId(...)`
- Persists `pendingTeamCreation` for Teams onboarding
- Writes subscription metadata on `subscription_data.metadata` including `tier` and `teamSize`

### 7.3 Webhook tier updates

**Route:** `app/api/stripe/webhook/route.ts` delegates to `lib/stripe-webhook-handlers.ts`.

Key behaviors:

- `handleSubscriptionUpdate` prefers **price id → tier** mapping, then metadata fallback (`normalizeTier`)
- For team tiers, updates `teams/{organizationId}` limits (`maxMembers`, etc.)
- Calls `syncEntitlementsForUser`

**Legacy note:** `handleSubscriptionCreated` defaults metadata tier to `"Pro"` if missing — risky if metadata is incomplete for non-Pro checkouts.

---

## 8) Firestore rules vs tier enforcement

`firebase/firestore.rules` generally **does not** encode subscription tiers. Access is mostly:

- org membership (`organizationId`, `teams/{id}.members`)
- personal database codes (`databaseCode`)
- tree sharing (`tree_permissions`)
- v3 workflow participant reads via `participantPrincipalIds` (for workflow collections)

**Implication:** “tier gating” is primarily enforced in **app routes** and **client UX**, not in rules (except indirectly via org boundaries).

---

## 9) Tier-adjacent “homeowner ↔ pro org” flows

These routes gate by **effective tier group** `home`/`plus` only:

- `POST /api/user/sync-default-pro-share` — `app/api/user/sync-default-pro-share/route.ts`
- `POST /api/user/revoke-default-pro-share` — `app/api/user/revoke-default-pro-share/route.ts`

These flows can create **CRM mirroring** on pro orgs (clients/properties) even though the homeowner may not have “Pro CRM” in the product matrix — it’s a **cross-tier sharing** mechanism.

---

## 10) Dev / beta paths that can distort tier state

### 10.1 Dev user factory

`app/api/dev/create-user/route.ts` can create synthetic users; for `teams` + subscription simulation it writes `subscription.tier: "Small Team"` (not `"Teams"`).

### 10.2 Teams free beta

`app/api/teams/create-free-beta/route.ts` sets:

- `subscription.tier: "Teams"`
- `subscription.status: "active"`

`getEffectiveSubscriptionTier` will treat this as **`Home`** unless `subscription.tier` normalizes to a team tier **and** status remains active/trialing. Because `"Teams"` normalizes to `Small Team`, it should still be effective **if** status is active.

**Operational risk:** any code path that leaves `subscription.tier` as `"Teams"` without normalization may still confuse UI labels; webhook updates should converge to concrete `Small Team` etc.

---

## 11) Findings (ranked)

### 11.1 Critical — participant workflow vs Teams-only drawer gating

- **Issue:** Participant Work (`participant-work` + participant list APIs) is designed for **v3 participants** (external/homeowner/etc.), but `view-request` / `view-quote` / `view-job` / `view-invoice` use `useTeamsOnlyAccess()` patterns that can block non-Teams users even when they are participants.
- **Evidence:** `lib/drawers.ts` (mixed `visibleForTiers`), `app/drawers/view-request.tsx` (early return for `showUpgradePrompt`), `app/drawers/participant-work.tsx` navigates to `view-*` drawers.
- **Impact:** broken UX for legitimate participant access; confusing “I have Work items but can’t open them”.

### 11.2 High — navigation truth vs capability truth drift

- **Issue:** `visibleForTiers` for some workflow *detail* drawers includes Home/Plus/Pro, while workflow *capabilities* remain Teams-only (`workflow.pipeline`).
- **Impact:** deep links / registry metadata imply access the capability system denies; upgrade prompts become the enforcement layer.

### 11.3 Medium — Stripe metadata defaults (`Pro` fallback)

- **Issue:** `handleSubscriptionCreated` defaults missing metadata tier to `Pro`.
- **Impact:** mis-tiering for edge cases / incomplete metadata.

### 11.4 Medium — ambiguous `"Teams"` tier string persistence

- **Issue:** some routes write `subscription.tier: "Teams"` while the app’s canonical tier strings are `Small Team` / `Mid Team` / `Large Team` / `Enterprise`.
- **Impact:** label drift, analytics ambiguity, and reliance on normalization + webhook updates to converge.

### 11.5 Low — documentation drift

- **Issue:** `docs/architecture/tiers-workflow-vs-record-keeping-audit.md` states clients-properties is Pro-only; current `lib/drawers.ts` includes Plus.
- **Impact:** internal confusion during refactors.

---

## 12) Recommended consolidation (engineering)

This is not a commitment to implement — it’s the lowest-risk direction to remove drift:

1. **Unify “workflow access” into three layers:**
   - **Capability:** org workflow pipeline (Teams)
   - **Participation:** `participantPrincipalIds` (v3 external access)
   - **Read-only vs mutate:** separate flags for org members vs participants
2. **Replace `useTeamsOnlyAccess()` gates in `view-*` workflow drawers** with:
   - `hasOrgWorkflow` (Teams pipeline) OR
   - `hasParticipantAccess` (based on loaded doc / server API)
3. **Align `visibleForTiers` with the matrix** (or generate it from a single table as described in `docs/architecture/unified-tier-workflow-system-plan.md`).

---

## 13) File index (quick)

- **Tier parsing + effective tier:** `lib/utils/tier-utils.ts`
- **Capabilities:** `lib/capabilities.ts`, `hooks/useCapabilities.ts`
- **Entitlements (server mirror):** `lib/groundzy/entitlements/*`
- **Limits:** `lib/ai-usage.ts`
- **Drawer registry:** `lib/drawers.ts`, `lib/drawer-registry.ts`
- **URL/tier guard:** `components/work-area/work-area-content.tsx`
- **Stripe mapping + webhook:** `lib/stripe-config.ts`, `lib/stripe-webhook-handlers.ts`, `app/api/stripe/*`
- **Participant work:** `app/drawers/participant-work.tsx`, `hooks/useParticipantWorkflow.ts`, `lib/firebase/participant-workflow.ts`, `app/api/workflow/participant-list/route.ts`, `lib/server/participant-workflow-list.ts`
- **Homeowner default-pro share tier gate:** `app/api/user/sync-default-pro-share/route.ts`, `app/api/user/revoke-default-pro-share/route.ts`

**v3 access model:** Workflow **record** access (view-request / view-quote / view-job / view-invoice) and server access envelopes are defined in [`docs/architecture/Groundzy v3 — Access & Permission System.md`](Groundzy%20v3%20—%20Access%20%26%20Permission%20System.md). Prefer that doc over §12’s historical “Teams vs participant” wording for current product rules.

---

*Generated from repository inspection on 2026-04-08.*
