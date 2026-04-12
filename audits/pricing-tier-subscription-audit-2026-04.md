# Groundzy pricing, tiers, and billing — current-state audit (April 2026)

**Scope:** Evidence from `app.groundzy`, `auth.groundzy`, and live `groundzy.com`. The `groundzy.com` workspace folder in this monorepo was **empty** (no source); marketing behavior is taken from the **live site** fetch (see appendix).

**Status labels used:** **Confirmed** (code or page directly shows it), **Likely** (strong inference), **Unclear** (needs Stripe dashboard / env / ops confirmation), **Legacy** (deprecated or duplicate paths).

---

## 1. Executive Summary

**What the product model is (Confirmed):**

- **Tiers** in the app are normalized strings such as `Home`, `Plus`, `Pro`, `Small Team`, `Mid Team`, `Large Team`, `Enterprise` (`lib/drawer-registry.ts` `SubscriptionTier`).
- **Paid access** for non-Home tiers is gated by **both** `subscription.tier` and `subscription.status`: `getEffectiveSubscriptionTier` only treats `Pro` and team tiers as paid if status is `active` or `trialing`; otherwise the user is treated as **Home** (`lib/utils/tier-utils.ts`).
- **Capabilities** for CRM and workflow are centralized in `lib/capabilities.ts` and mirrored server-side in `lib/groundzy/entitlements/resolve.ts` (workflow pipeline only for team tiers; CRM split Plus/Pro/Teams).
- **Usage limits** (AI chat, PlantNet / Identifying Wand, trees) are numeric per tier in `lib/ai-usage.ts`, enforced via API routes (e.g. `app/api/ai/chat/route.ts`, `app/api/trees/usage/route.ts`).

**Plans that exist in UX:**

- **Home** (free), **Home Plus** (paid add-on in auth onboarding and upgrade drawers), **Pro**, **Teams** (with **Small / Mid / Large** sizes; **Enterprise** as label + pricing mapping in code).
- Onboarding in **auth** speaks in **home / pro / teams** (`PlanTier` in `auth.groundzy/lib/pricing.ts`) before Firestore maps signup to `Home` / `Pro` / `Teams` in `createUserProfile` (`auth.groundzy/app/actions/auth.ts`).

**Biggest inconsistencies (Confirmed):**

1. **Two Stripe integrations:** `auth.groundzy` uses **inline `price_data`** + `lib/stripe-products.ts` (product IDs + cent amounts). `app.groundzy` uses **hard-coded Stripe price IDs** in `lib/stripe-config.ts`. Same Firestore `users` collection — **risk of conflicting checkout and webhook behavior** depending on which app handles a flow.
2. **Two webhook handlers:** `auth.groundzy/app/api/stripe/webhook/route.ts` and `app.groundzy/app/api/stripe/webhook/route.ts` implement **different** subscription logic (app side derives tier from **price ID** on update; auth side does **not**). **Unclear** which URL is registered in Stripe production; if both exist, duplicate or race conditions are possible.
3. **Plus tier handling on auth checkout:** Auth `create-checkout-session` sets subscription metadata `tier` to **`Home`** whenever `tier === "home"` — including **Home Plus** checkout (`auth.groundzy/app/api/stripe/create-checkout-session/route.ts`). The auth webhook’s `handleSubscriptionCreated` writes **`subscription.tier` from that metadata** (`auth.groundzy/lib/stripe-webhook-handlers.ts`). **Likely outcome:** paying Plus users coming only through auth may still have **`subscription.tier: "Home"`** unless a later event corrects it — **entitlement and marketing would disagree**.
4. **Marketing vs code:** Live `groundzy.com` FAQ claims **unlimited** Identifying Wand for Pro/Teams; `PLANTNET_USAGE_LIMITS` caps Pro at **500**/month (**Confirmed** mismatch).

**Intentional vs drift:**

- **Intentional:** `capabilities.ts` / `resolveEntitlementsFromUserDoc` alignment; drawer visibility lists (`lib/drawers.ts`) for nav; `getEffectiveSubscriptionTier` to block incomplete checkouts.
- **Drift / tech debt:** dual Stripe config, auth webhook normalization mapping **`plus` → `Pro`** in `handleSubscriptionUpdate` (`auth.groundzy/lib/stripe-webhook-handlers.ts`), enterprise mapped to Large Team pricing in `stripe-config.ts` (`getTeamTierName`).

---

## 2. Canonical Plan / Tier Inventory

| Name | Where it appears | Active / legacy / ambiguous | Notes |
|------|------------------|-------------------------------|--------|
| `Home` | Firestore `subscription.tier`, `SubscriptionTier`, onboarding | **Active** | Default free; `createUserProfile` sets `subscription.tier: "Home"` for `tier === "home"` (`auth/actions/auth.ts`). |
| `Plus` | `SubscriptionTier`, upgrade flows, `getPriceId` / Plus yearly | **Active** | **Not** a `PlanTier` in auth `CreateUserProfileTier` — introduced via Stripe / profile email path (`homeOption === "home_plus"` for welcome email only). |
| `Pro` | Signup, drawers, capabilities | **Active** | |
| `Teams` | Auth onboarding metadata / signup mapping | **Active** as **label** in Firestore for “teams” plan before team size resolution | Webhook + `normalizeTier` map **`Teams` / `teams` → `Small Team`** on read (`app/lib/utils/tier-utils.ts`). |
| `Small Team`, `Mid Team`, `Large Team`, `Enterprise` | `SubscriptionTier`, `PRICING_TIERS.teams.teamSizes`, Stripe price map | **Active** | `Enterprise` uses **Large Team** Stripe prices in `getTeamTierName` (**Confirmed** `lib/stripe-config.ts`). |
| `home`, `pro`, `teams` | Auth `PlanTier`, onboarding, API body | **Active** | Lowercase plan keys for API/UI. |
| `home_plus` | Auth `HOME_OPTIONS`, checkout body | **Active** | |
| Marketing “Groundzy Plus” / “Home Plus” | `groundzy.com`, `custom-pricing-table.tsx`, i18n | **Active** | |
| “Large Team” / “Enterprise” in FAQ | `groundzy.com` FAQ | **Active** copy | Enterprise described as 51+ users / contact sales; code maps enterprise size to Large Team pricing (**Confirmed** mismatch in positioning). |

---

## 3. Pricing Inventory

### Auth app (`auth.groundzy`) — display + checkout amounts

**File:** `lib/pricing.ts` (marketing-style labels)  
**File:** `lib/stripe-products.ts` (**checkout cents** + Stripe product IDs)

| Plan | Monthly (USD) | Yearly (USD) | Source | Notes |
|------|----------------|-------------|--------|--------|
| Home | 0 | 0 | `PRICING_PLANS.home` | |
| Pro | 19 | 171 | `PRICING_PLANS.pro` / `STRIPE_AMOUNTS.pro` | Cents: 1900 / 17100 |
| Teams (card headline) | 89 | 801 | `PRICING_PLANS.teams` | Per **TEAM_SIZES.small** |
| Home Plus | UI: `$9.99/year` style | `yearlyPrice: 9.99` in `HOME_OPTIONS` | `STRIPE_AMOUNTS.home_plus`: **monthly 999¢, yearly 999¢** | **Confirmed:** auth checkout uses **monthly or yearly** `unit_amount` from same cent table (`create-checkout-session/route.ts`). |
| Small Team | 89 | 801 | `TEAM_SIZES.small` / `STRIPE_AMOUNTS.small` | |
| Mid Team | 199 | 1791 | `TEAM_SIZES.mid` | |
| Large Team | 299 | 2691 | `TEAM_SIZES.large` | |

**Conflict:** `lib/pricing.ts` `HOME_OPTIONS.home_plus` lists `monthlyLabel: "$9.99/year"` while `monthlyPrice: 0` — display logic uses `yearlyAsMonthlyLabel` for yearly mode (`custom-pricing-table.tsx`). **Unclear** if monthly billing for Plus is intended; **app** checkout restricts Plus to **yearly** only (`getPriceId` in `stripe-config.ts`).

### App (`app.groundzy`) — Stripe price IDs

**File:** `lib/stripe-config.ts` — embedded **live/test** price IDs for Pro and Teams; Plus yearly via `STRIPE_PRICE_PLUS_YEARLY` or default `price_1T4U4X...`.

| Tier | Price IDs | Confirmed |
|------|-----------|-----------|
| Pro | `price_1Sr1YCHrFcfDYHeLIIWc0NE2` (monthly), `price_1Sr1YCHrFcfDYHeL3yh1pvkr` (yearly) | Code |
| Plus | Yearly only; env or default price ID | Code |
| Small/Mid/Large Team | Four pairs in `PRICE_TO_TIER` | Code |

**Single source of truth:** **None** — auth uses dynamic `price_data`; app uses **fixed price IDs**. **Revenue risk** if amounts diverge.

### Marketing site (`groundzy.com`) — **Confirmed** (fetched)

- Pricing section: Monthly/Yearly toggle; three columns align with Home / Pro / Teams narratives.
- Footer line: **“5 trees free • 20 AI Wizards/month • 10 AI Wand/month • Add-on: Groundzy Plus • No credit card required”** — aligns with `TREE_LIMITS` / `AI_USAGE_LIMITS` / `PLANTNET_USAGE_LIMITS` for **Home**.
- FAQ includes team **$89 / $199 / $299** monthly, annual discount claims, **seasonal seats** pricing, **enterprise** contact — **not verified** against enforcement in repo (partially **copy-only**).

---

## 4. Feature Entitlement Matrix

Legend: **SOT** = source of truth for enforcement.

| Capability | Home | Plus | Pro | Teams (any size) | SOT | UI | Backend |
|------------|------|------|-----|------------------|-----|-----|---------|
| Trees cap | 5 | 50 | unlimited (-1) | unlimited | `TREE_LIMITS` (`lib/ai-usage.ts`) | Map / add-tree flows use `tree-usage` | `app/api/trees/usage/route.ts` |
| AI chat / month | 20 | 50 | 500 | 500–5000 by tier | `AI_USAGE_LIMITS` | AI drawer | `app/api/ai/chat/route.ts` |
| PlantNet / Wand / month | 10 | 50 | 500 | 500–5000 | `PLANTNET_USAGE_LIMITS` | Wizards | `species/identify/reserve`, etc. |
| CRM **properties** | No | Yes | Yes | Yes | `capabilities.ts` | Drawer: Plus in `HOME_PLUS_PRO_AND_TEAMS`; properties drawers **Pro+ only** in `lib/drawers.ts` — **Confirmed:** properties list is **Pro + Teams**, not Plus-only | `resolveEntitlementsFromUserDoc`: `crmProperties` includes Plus |
| CRM **clients** | No | No | Yes | Yes | `capabilities.ts` | `clients-properties` / `clients` **Pro + Teams** only (`lib/drawers.ts`) | `crmClients`: Pro + Teams |
| Workflow (requests → invoices) | No | No | No | Yes | `capabilities.ts` / `can-append-event.ts` | Teams-only drawers | `assertCanAppendEvent` + `workflowPipeline` |
| Dashboard / weather / trees drawer (broad) | Yes | Yes | Yes | Yes | `visibleForTiers` | `HOME_PLUS_PRO_AND_TEAMS` | N/A |
| Hire a Pro drawer | Home, Plus | — | — | — | `hire-groundzy-pro` **['Home', 'Plus']** (`lib/drawers.ts`) | | |
| Map filters (properties/clients) | Home loses Pro filters | **Likely** Plus keeps | Pro | Teams | `mapbox-map.tsx` uses `getEffectiveSubscriptionTier` === `Home` to strip filters | | |

**Critical internal inconsistency (Confirmed):** `resolveEntitlementsFromUserDoc` grants **`crmProperties` to Plus**, but **`clients-properties` / `properties` drawers are hidden from Plus** (`visibleForTiers: ['Pro', ...]`). Server entitlements **do not match** primary nav for Plus for **property CRM surfaces**.

---

## 5. Upgrade / Downgrade Flow Analysis

| Flow | Start | Choices | System | After success | Gap |
|------|-------|---------|--------|---------------|-----|
| **Marketing → signup** | `groundzy.com` → `auth.groundzy.com/welcome` | Plans on pricing section link to welcome | Auth app | Onboarding multi-step including **step 4 = `CustomPricingTable`** (`onboarding/page.tsx`) | **Confirmed** |
| **Auth onboarding → Stripe** | Pricing step | home / pro / teams; Home Plus optional; billing cycle | `POST /api/stripe/create-checkout-session` on **auth** | Redirect to Stripe; success → `auth` payment success + app deep link | Metadata `tier` = `Home` for Home Plus (**risk**; see §1) |
| **In-app Plus** | Profile / upgrade UI | Plus yearly | `app` `create-checkout-session` with `tier: "Plus"` | Stripe | Metadata tier **Plus** (**Confirmed** `useProfileStripeActions.ts`) |
| **In-app Pro** | Profile | Pro monthly (default in hook) | `app` checkout | Stripe | |
| **Billing portal** | Profile | Manage subscription | `POST /api/stripe/create-portal-session` (**app**) | Stripe Customer Portal | **Unclear** if portal products cover all auth-created subscriptions |
| **Teams repair** | API | Manual repair | `auth` `app/api/teams/repair/route.ts` | Fixes org linkage | Partial / ops |
| **Downgrade** | Cancel in portal | — | Webhook sets `subscription.status: cancelled` | `getEffectiveSubscriptionTier` → **Home** for paid tiers | Plus with cancelled sub → treated as Home (**Confirmed** tier-utils) |

**Downgrade path:** No in-app “downgrade to Plus” ladder — user cancels; access falls back per status rules.

---

## 6. Stripe / Billing Architecture

### Checkout

- **Auth:** `stripe.checkout.sessions.create` with **`price_data`** (`auth/.../create-checkout-session/route.ts`). Metadata: `firebaseUserId`, `tier` (**Home/Pro/Teams** — not team size name), `billingCycle`, `teamSize`.
- **App:** Uses **existing price IDs** (`app/.../create-checkout-session/route.ts`); metadata includes raw `tier` string from client (e.g. `Plus`, `Pro`, `Teams`).

### Webhooks

- **Auth** (`auth/lib/stripe-webhook-handlers.ts`): `handleSubscriptionCreated` sets `subscription.tier` from metadata (default **`Pro`** if missing). **`Teams` team creation** via `createTeamForWebhook`. **`handleSubscriptionUpdate`** can set tier from metadata via **`normalizeTier` that maps `plus` → `Pro` (bug risk)**.
- **App** (`app/lib/stripe-webhook-handlers.ts`): `handleSubscriptionUpdate` **prefers `getTierFromPriceId`** for tier; syncs team doc limits; **`syncEntitlementsForUser`**. `handleSubscriptionCreated` uses metadata tier (default `Pro`) — **Teams** and org creation; sync entitlements.

### Subscription status

- **Active / trialing:** paid tier access (`tier-utils.ts`).
- **Incomplete / past_due / canceled:** effective tier **Home** for paid SKUs (except raw tier still visible for billing UX).

### Trials

- No dedicated “trial tier” in code; **`trialing`** Stripe status counts as active (**Confirmed**).

### Enterprise

- **Pricing:** `TeamSize` includes `'enterprise'`; maps to **Large Team** prices in `getTeamTierName` (**Confirmed**).
- **Team doc:** `Enterprise` → `maxMembers: 1000` in `getTeamLimitsFromTier` (**Confirmed** `stripe-webhook-handlers.ts`).

---

## 7. Access Enforcement Model

1. **Navigation / drawers:** `visibleForTiers` on each drawer (`lib/drawers.ts`) — **UI hiding**.
2. **Effective tier:** `getEffectiveSubscriptionTier(userDoc)` — **billing + status**.
3. **Capabilities:** `getCapabilitiesForTier` for product logic (`lib/capabilities.ts`).
4. **Server event pipeline:** `resolveEntitlementsFromUserDoc` + `assertCanAppendEvent` — **true enforcement** for workflow events (**Confirmed**).
5. **Usage limits:** Counts in Firestore / API — **enforcement** on AI and trees (**Confirmed** routes).
6. **Firestore rules:** Not fully audited here; **Unclear** if rules duplicate entitlements.

**Separation:** UI can hide features; **Plus vs Pro CRM** shows **entitlements and drawers can disagree** (§4).

---

## 8. Marketing vs Product Reality

| Topic | Marketing (`groundzy.com`) | Product | Verdict |
|-------|---------------------------|---------|---------|
| Free tier limits | 5 trees, 20 AI, 10 Wand (footer) | Same in `ai-usage.ts` | **Aligned** |
| Pro “unlimited” Wand | FAQ says unlimited for Pro/Teams | Caps at **500**/month | **Mismatch** |
| Plus | Add-on in footer | Paid tier with 50 trees, 50 AI, 50 PlantNet | **Partially aligned**; pricing **$9.99/yr** on site vs complex Stripe setup |
| Mid/Large “API / white-label / SLA” | FAQ claims | Not found as entitlement flags in repo | **Likely copy-only** |
| Enterprise | Contact sales | `Enterprise` tier + Large Team price mapping | **Partial** |
| Seasonal seats | FAQ with $/seat | `seasonalSeat` numbers in `PRICING_TIERS` only | **Unclear** enforcement |

---

## 9. Current User Journey by Persona

- **Homeowner (Home):** Free onboarding; **5 trees**, limited AI; Hire a Pro available; upgrade nudges to Plus/Pro/Teams drawers. **Confirmed** `lib/drawers.ts`.
- **Power homeowner (Plus):** Paid yearly (typical); **50 trees**; **drawer gating may block properties UI** while server entitlements suggest property CRM — **confusing** (**Confirmed** inconsistency).
- **Solo pro (Pro):** Pro checkout; full CRM clients+properties; no workflow pipeline without Teams. **Confirmed** capabilities.
- **Team/company:** Teams onboarding → org step → Stripe → webhook **team creation** (app path more complete). **Confirmed** webhook handlers.
- **Enterprise lead:** FAQ → **support@groundzy.com**; code may bill as Large Team. **Mismatch** risk.

---

## 10. Problems / Risks

### Product

- Plus value prop vs **nav exclusion** for properties.
- FAQ **unlimited** claims vs hard limits.

### UX

- Multiple upgrade entry points (auth vs app) with **different rules** (Plus yearly-only on app).
- Teams naming: **“Teams”** in Firestore vs **Small Team** in runtime.

### Technical

- **Dual** Stripe checkout + **dual** webhooks.
- Auth metadata **`Home` for Home Plus** checkout.
- Auth `normalizeTier` maps **plus → Pro** on subscription update.

### Revenue

- Price **drift** between `stripe-products` amounts and **live price IDs**.
- **Duplicate webhook** processing to same Firestore **Unclear** but high severity if misconfigured.

---

## 11. Evidence Appendix

| Claim | Evidence |
|-------|----------|
| Tier enum | `app.groundzy/lib/drawer-registry.ts` — `SubscriptionTier` |
| Effective access | `app.groundzy/lib/utils/tier-utils.ts` — `getEffectiveSubscriptionTier`, `isTierInGroup` |
| Capabilities | `app.groundzy/lib/capabilities.ts` |
| Entitlements | `app.groundzy/lib/groundzy/entitlements/resolve.ts`, `types.ts` |
| Usage limits | `app.groundzy/lib/ai-usage.ts` |
| Drawer gating | `app.groundzy/lib/drawers.ts` — `visibleForTiers`, `HOME_PLUS_PRO_AND_TEAMS`, `TEAMS_ONLY_TIERS` |
| Auth pricing display | `auth.groundzy/lib/pricing.ts`, `components/custom-pricing-table.tsx` |
| Auth checkout + amounts | `auth.groundzy/app/api/stripe/create-checkout-session/route.ts`, `lib/stripe-products.ts` |
| Auth webhook | `auth.groundzy/lib/stripe-webhook-handlers.ts`, `app/api/stripe/webhook/route.ts` |
| App checkout + price IDs | `app.groundzy/app/api/stripe/create-checkout-session/route.ts`, `lib/stripe-config.ts` |
| App webhook + portal | `app.groundzy/lib/stripe-webhook-handlers.ts`, `app/api/stripe/webhook/route.ts`, `app/drawers/profile/useProfileStripeActions.ts` |
| Signup tier constants | `app.groundzy/types/signup-flow.ts` — `PRICING_TIERS` |
| Profile creation mapping | `auth.groundzy/app/actions/auth.ts` — `subscriptionTier` |
| Workflow enforcement | `app.groundzy/lib/groundzy/policy/can-append-event.ts` |
| Marketing page | **Fetched** `https://groundzy.com` — pricing section, FAQ, footer limits |

---

## Answers (audit questions)

1. **Actual plans and prices:** Home (free); Plus (~$9.99/yr in UI, auth **999¢** in `STRIPE_AMOUNTS.home_plus`); Pro **$19/mo or $171/yr** (labels; cents in `stripe-products`); Teams **$89/$199/$299** monthly equivalents with yearly options — plus **Enterprise** priced as Large Team in app config. **No single price table in repo** matches all environments.

2. **What each plan unlocks:** See **§4**; trust **`capabilities.ts` + drawers + `ai-usage`** over marketing; **Plus** is ambiguous for **property CRM UI**.

3. **Upgrades and billing:** Auth onboarding checkout, in-app checkout on **app**, Stripe Customer Portal via **app** `create-portal-session`; webhooks on **both** apps possible — **ops-dependent**.

4. **Contradictions and revenue risks:** Dual Stripe pipelines; metadata/Home Plus bug on **auth**; FAQ vs **500** caps; Plus **entitlements vs drawers**.

5. **Cleanup before v3 pricing:** Consolidate **one** Stripe integration, **one** webhook, **one** price list; fix **Plus** tier metadata and **normalizeTier**; align **drawers** with `resolveEntitlementsFromUserDoc`; reconcile **marketing** FAQ with `ai-usage.ts`; document **Enterprise** vs Large Team.

---

*End of report.*
