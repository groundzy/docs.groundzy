# Groundzy — Marketing website strategy (evidence-based)

**Scope:** Grounding in `app.groundzy` (`C:\Groundzy\app`), `auth.groundzy` (`C:\Groundzy\auth`), and internal product docs. Where repos disagree or behavior is partial, gaps are called out explicitly.

---

## Ecosystem snapshot

### Surfaces

| Surface | Role (from code) |
|--------|-------------------|
| **auth.groundzy** | Signup slides (map + species), Firebase auth, onboarding (`/onboarding`), Stripe checkout (`/complete-payment`, `/payment/success`), invite (`/invite-code`), tier suggestion (`lib/onboarding-tier-suggestion.ts`), pricing display (`lib/pricing.ts`), shared tier model (`packages/pricing` → `RUNTIME_TIERS`, `CHECKOUT_PLANS`). |
| **app.groundzy** | Map-first shell (Mapbox), drawer UX, trees / zones / properties, CRM, AI (Wizard / Identifying Wand / chat) with monthly limits (`lib/ai-usage.ts`), workflow (requests → quotes → jobs → invoices) **only on team runtime tiers** (`lib/drawers.ts`, `lib/groundzy/entitlements/resolve.ts`), **Hire a Pro** for **Home** and **Plus** only (`visibleForTiers: ['Home', 'Plus']`), intelligence rules (e.g. storm × weak tree) + notification prefs copy. |

### Operational caveat

Internal audit material describes **two Stripe integrations** (auth vs app) and **two webhook handlers** with different behavior. Treat production billing as **Stripe dashboard / ops** truth, not assumed unified from code alone (`docs/audits/pricing-tier-subscription-audit-2026-04.md`).

---

## 1. Product positioning

### What Groundzy is

A **map-centric web app for tree care**: inventory and work with **trees** (and **zones** / **properties**) on a **Mapbox** map, structured records, **AI-assisted** surfaces with **tier-based monthly limits**. Paying users get **clients & properties**; **team runtime tiers** add **full workflow** (requests, quotes, jobs, invoices) and collaboration.

**References:** `docs/groundzy-v3-docs/01-product/groundzy-overview.md`, `lib/capabilities.ts`, `lib/groundzy/entitlements/resolve.ts`.

### Who it’s for

| Audience | Grounding |
|----------|-----------|
| **Homeowners & stewards** | `TIER_DESCRIPTIONS.home`, auth `PRICING_PLANS.home`; **Home** limits in `lib/ai-usage.ts` (e.g. **5 trees**, **20** AI messages/mo, **10** wand uses/mo). |
| **Plus** | More trees, higher limits, property tools (`lib/pricing.ts`); **Hire a Pro** still available (drawer **Home + Plus** only). |
| **Pro** | Unlimited markers/GZ-TIN-style access, unlimited wand/wizard in limits table, **clients & properties**, widgets, priority support (`types/signup-flow.ts` `PRICING_TIERS.pro`, `lib/pricing.ts`). **No full workflow pipeline** — workflow is **team tiers only**. |
| **Teams** (runtime bands) | Checkout `teams` → `Small Team` / `Mid Team` / `Large Team` / `Enterprise` (`packages/pricing/src/tier-model.ts`, `auth/app/actions/auth.ts`). Full workflow, collaboration, dashboards, admin, seasonal seats (`PRICING_TIERS.teams`). **SMS critical intelligence** — Teams-only per i18n (`lib/i18n/messages.ts`). |

### Core value proposition

1. **Map-first** — Single-page, map-first shell with drawers (`groundzy-overview.md`, `docs/groundzy-v3-docs/06-features/map.md`).
2. **Structured inventory** — Species, health, history, media (`docs/features/trees.md`).
3. **AI + identification** — Tiered limits (`lib/ai-usage.ts`).
4. **Property care → hire help** — **Hire a Pro** (`lib/drawers.ts`).
5. **Field work → ops** — CRM by tier (`lib/capabilities.ts`); **workflow** team-only (`lib/drawers.ts`, `resolve.ts`).

### Differentiators (code-backed)

- Map-first UX, drawer navigation, `?drawer=…` linking.
- **Explicit AI/wand caps** on lower tiers — good for honest marketing.
- **Workflow** only on **Small Team–Enterprise** (`lib/drawers.ts`).
- **Hire a Pro** for consumers only (`['Home', 'Plus']`).
- Intelligence / alerts: see **§7** before claiming a full “platform.”

**Avoid claiming without proof:** white-label, API, SLA as self-serve — Enterprise copy in `lib/pricing.ts` is quote-oriented; not evidenced as toggles in reviewed code.

---

## 2. Website architecture

### Sitemap (marketing domain)

Assumes public site (e.g. `www.groundzy.com`) separate from `app.` / `auth.`; CTAs → auth (`/signup`, `/onboarding`, `/complete-payment`) → app.

| URL | Purpose |
|-----|---------|
| `/` | Home |
| `/product/map-first-tree-care` | Map & inventory |
| `/product/ai-identification` | AI Wizard / Wand / limits |
| `/product/crm-properties` | Plus vs Pro (properties / clients) |
| `/product/workflow` | Teams-only pipeline (accurate scope) |
| `/solutions/homeowners` | Persona |
| `/solutions/professionals` | Persona |
| `/solutions/teams` | Persona |
| `/pricing` | Tiers + checkout |
| `/pricing/teams` | Seat bands + Enterprise CTA |
| `/compare/groundzy-vs-spreadsheets` | Content |
| `/resources/blog`, `/resources/blog/[slug]` | SEO |
| `/resources/guides/[slug]` | SEO |
| `/customers/case-studies/[slug]` | If assets exist |
| `/trust/security` | Privacy / data |
| `/legal/terms`, `/legal/privacy` | Legal |
| `/about`, `/contact` | Company |
| `/partners` | Only if program is real — verify |

**Auth routes (funnel):** `/`, `/welcome`, `/signup`, `/signin`, `/onboarding`, `/complete-payment`, `/payment/success`, `/invite-code`, `/forgot-password`, `/reset-password`.

### Page purposes

| Page type | Must achieve |
|-----------|----------------|
| **Home** | Category clarity; segment homeowner / pro / team; primary CTA signup. |
| **Product pillars** | Teach real modules (map, AI limits, CRM split, workflow scope). |
| **Solutions** | Persona outcomes using same facts. |
| **Pricing** | One tier story; bridge **checkout** (`home` / `plus` / `pro` / `teams`) vs **runtime** (`Home`, `Plus`, `Pro`, `Small Team`, …). |
| **Resources** | SEO + trust. |
| **Legal / trust** | Align with in-app disclaimers (e.g. intelligence not emergency monitoring). |

### URL notes

- Short static paths (`/pricing`, `/solutions/teams`).
- Don’t expose internal drawer IDs in URLs.
- One canonical pricing URL; campaigns via `?utm_*`.

---

## 3. Page-level breakdown

### `/` Home

| Section | Messaging intent | Conversion goal |
|--------|------------------|-----------------|
| Hero | Map-first tree care: inventory, plan, act. | Start free / View pricing |
| Segment chooser | Homeowner vs Pro vs Teams cards. | Route to solution or pricing |
| Product proof | Map + tree record + **limits** (`ai-usage.ts`). | Trust → signup |
| Workflow | Requests → quotes → jobs → invoices **as Teams story only**. | Teams pricing |
| Social proof | Logos/quotes **if available** — not in repo. | Optional |
| CTA | Free tier, no card (auth Home badge). | `auth` signup |

### `/product/map-first-tree-care`

| Section | Intent | Goal |
|--------|--------|------|
| Hero | Trees on the map — markers, zones, tools. | Signup |
| Deep dive | Trees, zones, properties; filters per `map-and-zones.md`. | Upgrade clarity |
| Limits | Home: **5 trees** (`TREE_LIMITS`). | Expectations |

### `/product/ai-identification`

| Section | Intent | Goal |
|--------|--------|------|
| Hero | Wizard + Wand + chat; not unlimited until Pro+ per `ai-usage.ts`. | Reduce churn |
| Comparison | Home vs Plus vs Pro caps. | Upgrade |

### `/product/crm-properties`

| Section | Intent | Goal |
|--------|--------|------|
| Hero | Properties vs clients **gated by tier** (`capabilities.ts`). | Right tier |
| Note | **Plus:** `crm.clients` false but `clients-properties` drawer includes Plus — **verify before “clients on Plus” claims.** | Internal alignment |

### `/product/workflow`

| Section | Intent | Goal |
|--------|--------|------|
| Hero | End-to-end pipeline for **team tiers** (`WORKFLOW_TIERS`). | Teams / sales |
| Clarify | **Pro** does **not** include this pipeline in entitlements. | No false expectations |

### `/pricing`

| Section | Intent | Goal |
|--------|--------|------|
| Hero | Home free; Plus; Pro; Teams + bands. | Checkout |
| Matrix | `PRICING_PLANS` + `ai-usage.ts` + capabilities. | Informed buy |
| Teams | `TEAM_SIZES` in `lib/pricing.ts` (seat ranges + prices). | Correct band |
| Enterprise | Quote; 51+ users (auth copy). | Sales |
| FAQ | Stripe duality — **don’t invent** unified billing until ops does. | Fewer tickets |

### Solutions

- **`/solutions/homeowners`** — Home, Plus, Hire a Pro, limits.
- **`/solutions/professionals`** — Pro CRM, export, widgets, AI unlimited per limits.
- **`/solutions/teams`** — Workflow, dashboards, admin, seasonal seats, SMS line (Teams).

---

## 4. SEO strategy

### Keyword clusters

| Cluster | Example terms | Tie to product |
|---------|---------------|----------------|
| Tree inventory / mapping | tree inventory software, property tree map | Map + trees |
| Arborist / ops | arborist software, tree care business software | CRM + workflow |
| Identification | tree identification app | AI / Wand |
| Property | property tree management | Plus/Pro properties |
| SMB workflow | tree service CRM, tree care quoting | Teams drawers |

### Page → intent (illustrative)

| Page | Primary intent |
|------|----------------|
| Home | Brand + “map-first tree care platform” |
| Map product | Tree map inventory, zones |
| AI product | Identification + assistant + “limits” |
| Workflow | Service workflow, invoicing (**Teams**) |
| Pricing | Groundzy pricing |
| Homeowners | Map + hire arborist |

### Content ideas

- Storm / tree risk guides (align with `lib/intelligence/storm-weak-tree.ts`; **informational** language per i18n).
- vs spreadsheets / generic CRM (map + workflow story).
- “What 5 trees / 20 AI messages means” (`ai-usage.ts`).
- Checkout `teams` → **Small Team** baseline in `createUserProfile` — long-tail “team plans.”

---

## 5. Conversion strategy

### Funnel

1. **Marketing** — Persona lanes.
2. **`/signup`** — Map + species slides (`auth/app/signup/page.tsx`).
3. **`/onboarding`** — `suggestTier` (`lib/onboarding-tier-suggestion.ts`).
4. **`/complete-payment`** — if paid tier `incomplete` (`auth/app/page.tsx` + `hasPaidTierIncomplete`).
5. **App** — Map shell; activation = first tree / property / workflow step by persona.

### CTAs

| Context | CTA |
|---------|-----|
| Home (homeowner) | Start free — map your trees |
| Home (pro) | Go Pro — only if copy matches `ai-usage` + capabilities |
| Home (team) | See Teams pricing / Book demo (Enterprise) |
| Pricing | Tier cards → auth checkout (`getStripePriceId` / `lib/pricing.ts`) |
| Workflow page | Upgrade to Teams (mirrors in-app drawer) |

### Trust

- Publish **numeric limits** from `lib/ai-usage.ts`.
- **Alert disclaimers:** informational, not emergency monitoring (`lib/i18n/messages.ts`).
- **Case studies / reviews:** not in repo — omit or “coming soon.”

### Friction reduction

- Pricing: monthly anchor; yearly ≈ 10× monthly (`lib/pricing.ts` header).
- **Plus** naming (not “Home Plus” except legacy `HOME_OPTIONS`).
- Onboarding: role / goal / tree count **suggests** tier.

---

## 6. Tier & pricing clarity

### Definitions

- **Checkout plans:** `home` \| `plus` \| `pro` \| `teams` (`packages/pricing/src/tier-model.ts`).
- **Runtime tiers:** `Home`, `Plus`, `Pro`, `Small Team`, `Mid Team`, `Large Team`, `Enterprise`.
- **Teams** checkout maps toward **Small Team** baseline in `createUserProfile`; refine at Stripe/seat selection — marketing: *“Teams start at Small Team; pick seat band at checkout.”*

### Conflicts before publishing

| Issue | Detail |
|-------|--------|
| **Plus in app struct** | `types/signup-flow.ts` `PRICING_TIERS` omits **Plus**; **auth** includes it — prefer **auth + `@groundzy/pricing`**. |
| **Enterprise** | Auth: 51+ / custom; app Stripe `getTeamTierName` maps `enterprise` size → **Large Team** prices — don’t promise a separate Enterprise SKU without ops. |
| **Plus billing** | Older docs said yearly-only; `lib/stripe-config.ts` has monthly path for Plus — treat as **monthly + yearly** unless Dashboard says otherwise. |
| **Pro vs workflow** | **No** full request→invoice pipeline on Pro (`workflowPipeline` team-only). |

### Simplify presentation

1. Four columns: **Home**, **Plus**, **Pro**, **Teams** (+ seat selector); **Enterprise** = contact sales.
2. Explain **Small / Mid / Large** = runtime names on receipts.
3. One row: trees / AI / wand from `lib/ai-usage.ts`.
4. Footnote: *Full workflow requires a Teams plan.*
5. Ops: unify Stripe/webhook story before “one billing portal” copy.

---

## 7. Differentiation & narrative

### Defensible angles

- Map + arboriculture data model (trees, zones, properties, history).
- Clear **tier split**: hire help (Home/Plus) vs **CRM** (Pro) vs **pipeline** (Teams).
- **AI limits** visible — comparison-friendly.
- **Hire a Pro** only on consumer side — coherent.

### Headlines (test)

- “Tree care starts on the map.”
- “Inventory, insight, and operations—in one map-first workspace.” (Prefer “tools & alerts” if intelligence is still partial.)

### Taglines (test)

- “Map-first tree care.”
- “From canopy to customer—in one place.”

### Story arc

1. **Problem** — Fragmented maps, photos, spreadsheets, inbox.
2. **Solution** — Every tree in geospatial context + records; CRM or workflow by tier.
3. **Outcome** — Homeowners plan & hire; pros manage clients/properties; teams run the pipeline.

### Intelligence (careful)

| Status | What |
|--------|------|
| **In code** | e.g. storm × weak tree evaluation; email/SMS prefs (SMS Teams). |
| **Spec / partial** | Full alert engine (`docs/groundzy-v3-docs/07-systems/alert-notification-system.md`). |
| **Marketing** | “Timely, informational insights — not emergency monitoring.” No guaranteed 24/7 delivery. |

---

## 8. Inconsistencies & gaps (explicit)

| Topic | Finding |
|-------|---------|
| **Plus in app `PRICING_TIERS`** | Missing in `types/signup-flow.ts`; use auth + `@groundzy/pricing`. |
| **Pro vs workflow** | Don’t imply full workflow on Pro — `workflow.pipeline` is team tiers. |
| **Plus vs clients** | Capabilities: no `crm.clients` on Plus; drawer UX may blur — clarify before claims. |
| **Stripe** | Two implementations — avoid “single source of truth” for billing. |
| **Enterprise** | Custom UX vs Large Team price mapping — don’t overpromise distinct SKUs. |
| **Social proof** | No customer counts/reviews in repo — omit or verify offline. |

---

*This structure supports a cohesive **Groundzy v3** story—map-first operations, transparent limits, persona paths, and **Teams as the business layer**—aligned with what **auth** and **app** implement today.*
