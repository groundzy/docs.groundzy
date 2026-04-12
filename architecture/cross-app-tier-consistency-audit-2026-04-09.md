# Groundzy cross-app tier consistency audit — 2026-04-09

## 1. Executive summary

Across `groundzy.com`, `auth.groundzy`, and `app.groundzy`, **pricing amounts and yearly/monthly framing are largely aligned in code** (e.g. Pro **$19/mo · $190/yr**, Small Team **$89/mo · $890/yr**, Plus **$9/mo · $90/yr** fallbacks). **Firestore runtime tiers** are a small set of **Title Case** strings (`Home`, `Plus`, `Pro`, `Small Team`, `Mid Team`, `Large Team`, `Enterprise`), while **checkout** uses lowercase **plan keys** (`home` | `pro` | `teams`) plus **`home_plus`** and **`teamSize`** (`small` | `mid` | `large`).

The **largest cross-surface tension** is **workflow positioning**: the marketing site and shared signup copy describe **full business workflow** (requests → invoices) as **“Everything in Pro, plus …”** under **Teams**, which reads as **Teams-only**. The **app** implements **Pro solo workflow** as well: global workflow list drawers include **Pro**, creation policy allows **Pro** when the org scope is the solo user, and **`workflowPipeline` entitlement is Teams-only** but **`canUseWorkflowListSurface`** is **Teams pipeline OR Pro**. Product and marketing should explicitly decide whether **Pro** is allowed to sell a **solo pipeline**; today **code allows it**, **copy often implies it does not**.

Secondary risks: **Stripe subscription metadata** uses the literal string **`Teams`** for team checkouts while **Firestore** stores **Small/Mid/Large Team** (resolution relies on **price IDs** and **`teamSize`** metadata); **`resolveFirestoreTierFromSubscription`** defaults missing metadata to **`Pro`**; **`types/signup-flow.ts`** still documents **legacy team size keys** (`10` / `25` / `50`) alongside **`enterprise`**, while **auth checkout** uses **`small` / `mid` / `large`** (mapped in `app.groundzy/lib/stripe-config.ts`). **Plus** is **not** a top-level `PlanTier` in auth—it is **`tier: "home"` + `homeOption: "home_plus"`**.

This document is **evidence-based from repository code as of the audit date**; it does **not** assert Stripe Dashboard or production env overrides beyond what is visible in code and committed defaults.

---

## 2. Canonical tier and plan inventory

### 2.1 Runtime subscription tiers (Firestore / app / auth)

**Canonical strings** (typed in app and auth):

| Exact string | Where | Purpose |
|--------------|--------|---------|
| `Home` | `app.groundzy/lib/drawer-registry.ts` (`SubscriptionTier`), `auth.groundzy/lib/tier-utils.ts` | Default/free; effective tier when paid status inactive |
| `Plus` | Same | Paid add-on on Home (Stripe prices); properties CRM tier; not a `PlanTier` in auth onboarding |
| `Pro` | Same | Solo paid professional |
| `Small Team`, `Mid Team`, `Large Team` | Same | Paid team tiers (seat bands in `auth.groundzy/lib/pricing.ts` → `TEAM_SIZES`) |
| `Enterprise` | Same | Declared in tier unions; **no default Stripe price→tier row** in `auth.groundzy/lib/stripe-price-ids.ts` (likely sales / custom) |

**Ambiguous / bridging strings (non-canonical for Firestore tier):**

| String | File(s) | Role |
|--------|---------|------|
| `Teams` | `auth.groundzy/app/api/stripe/create-checkout-session/route.ts` (`subscription_data.metadata.tier`), `auth.groundzy/lib/stripe-webhook-tier.ts` | **Checkout metadata label** for any team purchase; **not** stored as final tier if price ID resolves |
| `teams` / `Teams` | `app.groundzy/lib/utils/tier-utils.ts` `normalizeTier` | Maps **`teams` / `Teams` → `Small Team`** (default team size) |
| `home`, `pro`, `teams` | `auth.groundzy/lib/pricing.ts` `PlanTier` | **Checkout / UI plan keys** (lowercase) |
| `home_plus` | `auth.groundzy/lib/pricing.ts` `HomeOptionKey` | **Plus** checkout path |

### 2.2 Marketing labels (groundzy.com)

| Label | Example location | Notes |
|-------|------------------|--------|
| `HOME`, `PRO`, `TEAMS` | `groundzy.com/index.html` pricing cards | Display-only uppercase labels |
| `Groundzy Plus` | `index.html` add-on line, FAQ | Add-on, not a fourth column |
| `Small Team`, `Mid Team`, `Large Team`, `Enterprise` | FAQ text in `index.html` | Team **names** + dollar amounts |

### 2.3 Entitlement / capability labels (app)

| Label | File | Role |
|-------|------|------|
| `workflow.pipeline` | `app.groundzy/lib/capabilities.ts`, `hooks/useCapabilities.ts` | **True only for team tiers** in tier-derived capabilities |
| `legacy.pro_plus_teams_surfaces` | `lib/capabilities.ts` | Bundle for Pro/Plus/Teams **UI surfaces** (not org pipeline) |
| `crm.clients`, `crm.properties` | `lib/capabilities.ts`, `lib/groundzy/entitlements/resolve.ts` | CRM gates |

### 2.4 Stripe / billing identifiers

| Kind | File | Purpose |
|------|------|---------|
| Price IDs → tier | `auth.groundzy/lib/stripe-price-ids.ts`, `app.groundzy/lib/stripe-config.ts` | **Source of truth for tier from subscription line item** |
| Checkout price selection | `auth.groundzy/lib/stripe-checkout-prices.ts` | Env overrides with committed defaults |
| Product IDs + cent amounts | `auth.groundzy/lib/stripe-products.ts` | Fallback `price_data` when price ID path not used |

---

## 3. Marketing site findings (`groundzy.com`)

### 3.1 Pricing presentation

- **Files:** `index.html` (structure, copy, schema.org), `css/pricing.css`, `modules/landing.js` (`initPricingBillingToggle`, `updatePricingDisplay`).
- **Billing toggle:** Monthly vs yearly buttons; JS sets Pro to **$19/month** or **$190/year**, Small Team display to **$89/month** or **$890/year** (`modules/landing.js`).
- **CTAs:** Primary links point to **`https://auth.groundzy.com/welcome`** for Get Started / Get Pro / Create Team (`index.html`).
- **Hero/footer limits line:** “**5 trees free • 20 AI Wizards/month • 10 AI Wand/month • Add-on: Groundzy Plus**” — aligns with **`TREE_LIMITS` / `AI_USAGE_LIMITS` / `PLANTNET_USAGE_LIMITS`** for **Home** in `app.groundzy/lib/ai-usage.ts`.

### 3.2 Tier naming and grouping

- Three main columns: **Home**, **Pro**, **Teams** (TEAMS header). **Plus** is described as an **add-on** under Home, not a separate plan column.
- **Teams** card shows **“Starting at”** and **Small Team: 1–5 users** badge; Mid/Large are **FAQ-only** on the main pricing grid.

### 3.3 Feature / workflow promises (copy vs app)

- **Teams** plan bullet list: “**Everything in Pro, plus:**” then **full workflow** (requests, quotes, jobs, invoices), collaboration, dashboards, admin, account management, seasonal seats (`index.html`).
- **Interpretation:** This **implies Pro does not include the full workflow block**. In **`app.groundzy`**, **Pro** users **do** get workflow **list** drawers and **solo** create access per policy (see §5 and §7). This is a **positioning mismatch**, not a pricing-number mismatch.

### 3.4 Structured data (schema.org)

- **SoftwareApplication** `offers` includes Home (0), Pro (19), Small/Mid/Large team prices (`index.html` JSON-LD). Mid/Large omit `priceSpecification` in one block — **less structured** than Pro/Small.
- Internal doc `groundzy.com/MARKETING_PAGE_AUDIT.md` flags historical **FAQ vs schema** inconsistencies; **current `index.html` excerpt** shows **non-zero** prices for team offers — still treat **FAQ vs UI vs Stripe** as separate channels if editing.

### 3.5 Enterprise

- FAQ: **51+ users**, contact sales (`index.html`). **No** self-serve Enterprise price on the page. **`Enterprise` exists as a tier enum** in app/auth but **Stripe default maps** in code point **enterprise → Large Team pricing** in `app.groundzy/lib/stripe-config.ts` (`getTeamTierName`).

---

## 4. Signup / auth / checkout findings (`auth.groundzy`)

### 4.1 Plan model for onboarding

- **`PlanTier`:** `"home" | "pro" | "teams"` (`auth.groundzy/lib/pricing.ts`).
- **Plus:** `HomeOptionKey`: `"home" | "home_plus"`; checkout requires **`tier === "home"`** and **`homeOption === "home_plus"`** for paid Home (`auth.groundzy/app/api/stripe/create-checkout-session/route.ts`).
- **Teams:** `TeamSizeKey`: `"small" | "mid" | "large"` (`lib/pricing.ts` / `lib/stripe-products.ts`).

### 4.2 Display pricing (auth)

- **`PRICING_PLANS` and `TEAM_SIZES` and `HOME_OPTIONS`** (`auth.groundzy/lib/pricing.ts`) match marketing figures: Pro **19 / 190**, teams **89/199/299** by size, Plus **9 / 90**, yearly savings helper **`yearlySavingsDollars`**.
- **UI:** `auth.groundzy/components/custom-pricing-table.tsx` consumes those constants.

### 4.3 Checkout API behavior

- **POST body:** `tier`, `billingCycle`, `teamSize`, `homeOption`, optional `organizationData`, `returnUrl`, etc. (`create-checkout-session/route.ts`).
- **Stripe `subscription_data.metadata`:** `tier` is set to **`Plus`**, **`Pro`**, **`Home`**, or **`Teams`** string — **not** `Small Team` etc. (`create-checkout-session/route.ts`). **`teamSize`** is **`small|mid|large`** for team checkouts, else **`"1"`**.
- **Firestore tier after webhook:** from **`resolveFirestoreTierFromSubscription`** (`auth.groundzy/lib/stripe-webhook-tier.ts`): prefers **price ID** mapping; else metadata; **`Teams` + `teamSize`** → **`teamSizeKeyToFirestoreTier`**.

### 4.4 Risky default (webhook)

- If **`getTierFromPriceId`** returns null **and** metadata is empty, **`resolveFirestoreTierFromSubscription` returns `"Pro"`** (`stripe-webhook-tier.ts`). **Dangerous** if a new price ID is not added to **`stripe-price-ids.ts`** in lockstep.

### 4.5 Post-checkout entitlements (auth)

- **`lib/entitlements/sync-user.ts`** recomputes **`workflowPipeline`** for **team tiers only**; **Plus** gets **`crmProperties`** but **not** **`crmClients`**; **Pro** gets **clients + properties**; aligns with **`lib/capabilities.ts`** rules in app when tier-derived.

### 4.6 Auth vs app duplicate tier helpers

- **`auth.groundzy/lib/tier-utils.ts`** and **`app.groundzy/lib/utils/tier-utils.ts`** both implement **effective tier** (paid only if **active/trialing**). **Plus** handling: auth **`getEffectiveSubscriptionTier`** treats Plus like other paid tiers (needs active status). **App** `getEffectiveSubscriptionTier` only special-cases **Home** before status check — behavior is **consistent** for gating paid tiers.

---

## 5. App runtime findings (`app.groundzy`)

### 5.1 Tier source and “effective” tier

- **Raw:** `subscription.tier` on user doc (`getUserSubscriptionTier` in `lib/utils/tier-utils.ts`).
- **Effective:** **`getEffectiveSubscriptionTier`**: non-active subscription → **`Home`** for paid tiers (so incomplete checkout does not unlock paid features).

### 5.2 Capabilities vs persisted entitlements

- **Client UI:** `useCapabilities()` → **`getCapabilitiesForTier(tier)`** (`hooks/useCapabilities.ts`, `lib/capabilities.ts`) — **purely tier-derived**, does **not** read `userDoc.entitlements` fields.
- **Server / rules alignment:** `lib/groundzy/entitlements/resolve.ts` uses the **same rules** as `capabilities.ts` for **`workflowPipeline`**, CRM flags, etc.
- **Important:** **`workflowPipeline` entitlement is `true` only for team tiers** in both places; **Pro** workflow creation is handled separately in **`lib/groundzy/policy/access/can.ts`** (`Pro` + org id === uid).

### 5.3 Global workflow list surface (Pro vs Teams)

- **`useGlobalEntitlements`** (`hooks/useGlobalEntitlements.ts`): **`canUseWorkflowListSurface = workflowPipeline || tier === "Pro"`**.
- **Drawer registration** (`lib/drawers.ts`): **`requests` / `quotes` / `jobs` / `invoices`** use **`TEAMS_AND_PRO_TIERS`** (Pro + all team tiers). **Add** forms for workflow entities are **`TEAMS_ONLY_TIERS`** — **Pro** uses **different** creation paths (policy), not necessarily these drawers — **record-level** behavior is split across **`useWorkflowResourceAccess`**, policy, and participant flows (v3 files under `hooks/`, `lib/groundzy/policy/`). **Do not flatten** to “Teams only” without reading those files.

### 5.4 Usage limits (numeric enforcement)

- **`lib/ai-usage.ts`:** **Home:** 5 trees, 20 AI, 10 PlantNet; **Plus:** 50 / 50 / 50; **Pro+:** unlimited for AI/PlantNet, unlimited trees (`-1`).
- **APIs:** e.g. `app/api/trees/usage`, `app/api/ai/usage`, `app/api/ai/chat`, `app/api/species/identify/reserve` (see existing internal `docs/architecture/tier-system-audit-2026-04-08.md` for route list).

### 5.5 Hire a Pro (product boundary)

- **Drawer `hire-groundzy-pro`:** **`visibleForTiers: ['Home', 'Plus']`** only (`lib/drawers.ts`). **Pro** users **do not** get this drawer — consistent with “homeowner hires a contractor” positioning.

### 5.6 CRM drawers

- **`clients-properties` / `properties`:** **`Plus`, `Pro`, team tiers** (`lib/drawers.ts`).
- **`crmClients` capability:** **Pro + teams only** (`lib/capabilities.ts`) — **Plus** sees properties surfaces but **not** client CRM entitlement at capability layer.

---

## 6. Stripe and billing mapping

### 6.1 Price ID → Firestore tier

- **Auth:** `auth.groundzy/lib/stripe-price-ids.ts` — maps each **committed default** price id to **`Pro`**, **`Small Team`**, **`Mid Team`**, **`Large Team`**, or **Plus** (via **`plusPriceIds()`** list).
- **App (duplicate map):** `app.groundzy/lib/stripe-config.ts` **`PRICE_TO_TIER`** + **`getTierFromPriceId`** — **must stay aligned** with auth (comments reference cross-repo alignment).

### 6.2 Checkout line items

- **Preferred path:** fixed **Price IDs** from **`getStripeCheckoutPriceId`** (`auth.groundzy/lib/stripe-checkout-prices.ts`).
- **Fallback:** **`price_data`** with **`STRIPE_PRODUCTS`** + **`STRIPE_AMOUNTS`** cents (`auth.groundzy/lib/stripe-products.ts`): Pro **1900/19000**, Plus **900/9000**, team sizes **8900/89000**, etc. — **matches** marketing **dollar** amounts.

### 6.3 Metadata and team resolution

- Checkout sets **`tier: "Teams"`** for all team purchases; **webhook** resolves to **`Small Team` / `Mid Team` / `Large Team`** using **price id** or **`teamSize`** (`auth.groundzy/lib/stripe-webhook-tier.ts`).
- **Display vs storage:** Users see **“Teams”** only as **marketing/auth grouping**; **Firestore** should hold **concrete** team tier names for runtime.

### 6.4 Legacy / dual key systems

- **`app.groundzy/types/signup-flow.ts`:** **`TeamSize`** `'10'|'25'|'50'|'enterprise'` and **`PRICING_TIERS.teams.teamSizes`** — **parallel** to auth’s **`small|mid|large`** (`TEAM_SIZES`).
- **`app.groundzy/lib/stripe-config.ts` `getTeamTierName`** maps **`10→Small Team`**, **`25→Mid`**, **`50→Large`**, **`enterprise→Large Team`**. **Risk:** two naming schemes in one product codebase.

### 6.5 Comment drift

- **`app.groundzy/lib/stripe-config.ts`** comment on Plus yearly references **“$9.99/year”** while env default id is paired with **9000 cents** fallback elsewhere — treat comment as **suspect**; **trust `STRIPE_AMOUNTS` / `pricing.ts`**.

---

## 7. Feature promise vs actual enforcement

| Topic | Marketing / signup copy (evidence) | Auth / Stripe | App enforcement (evidence) |
|-------|-----------------------------------|---------------|----------------------------|
| **CRM / clients / properties** | Pro: “Clients & properties”; Home FAQ: Plus “property-focused tools” (`groundzy.com/index.html`, `auth.groundzy/lib/pricing.ts`) | Plus: **`crmProperties` only** in `auth.groundzy/lib/entitlements/sync-user.ts` | **Plus:** properties drawers + **`crm.properties`**; **not** **`crm.clients`** (`lib/capabilities.ts`). **Pro/teams:** both CRM capabilities where stated |
| **Workflow pipeline** | Teams card: “Everything in Pro, plus” **full workflow** (`index.html`) | **`workflowPipeline`** true **only** for team tiers in sync-user | **Teams:** `workflowPipeline`; **Pro:** **solo** create allowed via **`canCreateWorkflowEntity`**; **lists** show for **Pro** (`useGlobalEntitlements`, `lib/drawers.ts`) — **copy undersells Pro** |
| **Participant / record views** | Not detailed on marketing | N/A | **v3** access in `hooks/useWorkflowResourceAccess.ts`, `lib/groundzy/policy/access/`, `app/drawers/participant-work.tsx` — **separate** from tier column copy |
| **AI / Wand / trees** | Home limits + Plus upsell; Pro “unlimited” (`index.html`, FAQ) | Plus checkout amounts | **`lib/ai-usage.ts`** numeric caps; APIs enforce |
| **Hire a Pro** | Plus add-on mentions Hire a Pro (`index.html`) | N/A | Drawer **Home + Plus only** (`lib/drawers.ts`) |
| **Teams collaboration** | FAQ Teams (`index.html`) | Team org creation in webhook (`stripe-webhook-handlers.ts`) | **Team** drawer, org-scoped features |
| **Map / job layers** | FAQ implies team features | **`mapLayersJobs`** teams only in entitlements sync | **`map.layers_jobs`** capability for team tiers (`lib/capabilities.ts`) |
| **Dashboard / admin** | “Team dashboards”, admin (`index.html`) | Metadata / org | Mixed; **weather** tiers use **`getWeatherCardTier`** (`lib/utils/tier-utils.ts`) |

---

## 8. Cross-app inconsistencies and risks

1. **Workflow narrative:** Marketing **Teams** column and FAQ **list workflow under Teams**; **app** grants **Pro** significant workflow **surface and solo policy**. Either **update marketing** to describe **Pro solo workflow** or **narrow app** — **cannot recommend** without product decision; **code currently supports Pro workflow**.
2. **`Teams` metadata vs Firestore tier:** Checkout metadata **`Teams`** vs stored **`Small Team`**, etc. — **reliant** on **price IDs** and **`teamSize`**; **failure mode** if misconfigured.
3. **Webhook default to `Pro`:** Missing price map + missing metadata → **`Pro`** (`auth.groundzy/lib/stripe-webhook-tier.ts`).
4. **`normalizeTier` maps `teams` → `Small Team`:** Could **mask** bad data instead of failing visible.
5. **Dual team size vocabularies:** **`small/mid/large`** vs **`10/25/50/enterprise`** (`types/signup-flow.ts` vs `auth` pricing).
6. **Enterprise:** Sold as custom in FAQ; **code** maps **enterprise → Large Team** price in **`stripe-config.ts`** — **clarify** actual billing for Enterprise accounts.
7. **Internal docs:** Older audits (e.g. `docs/audits/pricing-tier-subscription-audit-2026-04.md`) may cite **stale** numeric caps; **`lib/ai-usage.ts`** is the **numeric SOT** for enforcement today.

---

## 9. Recommended canonical tier model

**Purpose:** single cross-app vocabulary for **engineering + product**. This is a **recommendation** to align naming; **verify** with Stripe and GTM before renaming anything user-facing.

### 9.1 Canonical runtime tier names (Firestore / app)

Keep **exactly:** **`Home` | `Plus` | `Pro` | `Small Team` | `Mid Team` | `Large Team` | `Enterprise`**.

- **Never** persist **`Teams`** as a user’s `subscription.tier` — use **concrete** team tier or **`Enterprise`**.

### 9.2 Canonical checkout / plan keys (auth API)

- **`tier`:** `home` | `pro` | `teams`
- **`homeOption`:** `home` | `home_plus` (Plus)
- **`teamSize`:** `small` | `mid` | `large`
- **`billingCycle`:** `monthly` | `yearly`

### 9.3 Canonical marketing group names

- **Column names:** **Home**, **Pro**, **Teams** (with **Groundzy Plus** as **Home add-on**).
- **Team size names:** **Small / Mid / Large Team**, **Enterprise** (contact sales).

### 9.4 Canonical Stripe mapping

- **SOT for price id → tier:** **`auth.groundzy/lib/stripe-price-ids.ts`** (deployed webhook path); **`app.groundzy/lib/stripe-config.ts`** should **mirror** or **share** a single package in the future.
- **SOT for checkout price pick:** **`auth.groundzy/lib/stripe-checkout-prices.ts`** + env **`STRIPE_PRICE_*`**.

### 9.5 Canonical display rules

- **Marketing static site:** `groundzy.com/index.html` + `modules/landing.js` (toggle).
- **Auth onboarding pricing:** `auth.groundzy/lib/pricing.ts` + `components/custom-pricing-table.tsx`.
- **Runtime feature matrix (enforcement):** `app.groundzy/lib/capabilities.ts`, `lib/groundzy/entitlements/resolve.ts`, `lib/ai-usage.ts`, `lib/drawers.ts`.

### 9.6 Strings to treat as legacy or internal-only

- **`Teams` as metadata** — internal to Stripe session only.
- **`10` / `25` / `50` team keys** in **`types/signup-flow.ts`** — migrate docs and consumers to **`small` / `mid` / `large`** or document mapping explicitly.
- **“$9.99/year”** style comments — remove or align with **cents** in `STRIPE_AMOUNTS`.

---

## 10. File index / evidence sources

### `groundzy.com`

- `index.html` — pricing cards, FAQ, CTAs to `auth.groundzy.com`, schema.org offers + FAQPage
- `modules/landing.js` — monthly/yearly price display for Pro and Small Team
- `css/pricing.css` — pricing section styling
- `MARKETING_PAGE_AUDIT.md` — internal notes on schema/FAQ (verify against current `index.html`)

### `auth.groundzy`

- `lib/pricing.ts` — `PlanTier`, `PRICING_PLANS`, `HOME_OPTIONS`, `TEAM_SIZES`, display labels
- `lib/stripe-checkout-prices.ts` — checkout price ID selection
- `lib/stripe-price-ids.ts` — price id → Firestore tier
- `lib/stripe-products.ts` — product ids + fallback cent amounts
- `lib/stripe-webhook-tier.ts` — subscription → Firestore tier + **defaults**
- `lib/stripe-webhook-handlers.ts` — Firestore user update, team creation, **`syncEntitlementsForUser`**
- `lib/tier-utils.ts` — subscription tier types, effective tier, **`isTierInGroup`**
- `lib/entitlements/sync-user.ts` — webhook entitlements mirror rules
- `app/api/stripe/create-checkout-session/route.ts` — checkout session + **metadata**
- `components/custom-pricing-table.tsx` — onboarding pricing UI

### `app.groundzy`

- `lib/utils/tier-utils.ts` — effective tier, **`normalizeTier`**, weather mapping
- `lib/drawer-registry.ts` — **`SubscriptionTier` union**
- `lib/drawers.ts` — **`visibleForTiers`**, workflow tier sets, Hire a Pro
- `lib/capabilities.ts` — capability matrix from tier
- `lib/groundzy/entitlements/resolve.ts` — server entitlements from tier
- `lib/groundzy/policy/access/can.ts` — **Pro solo** workflow create rule
- `lib/ai-usage.ts` — tree / AI / PlantNet limits
- `lib/stripe-config.ts` — price id map, Plus ids, legacy team size map
- `hooks/useGlobalEntitlements.ts`, `hooks/useCapabilities.ts`, `hooks/useTeamsOnlyAccess.ts` — runtime UX gates
- `types/signup-flow.ts` — legacy **`PRICING_TIERS`** + old team size keys

---

*End of audit — 2026-04-09.*
