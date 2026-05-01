# Groundzy Pricing Reference

Consolidated reference across all three repos: **auth.groundzy** (onboarding), **groundzy.com** (marketing), and **app.groundzy** (the app). Covers tier definitions, feature gates, limits, upgrade prompts, and discrepancies between surfaces.

---

## Tiers at a Glance

| | Home | Plus | Pro | Small Team | Mid Team | Large Team | Enterprise |
|---|---|---|---|---|---|---|---|
| **Price (monthly)** | Free | $9 | $19 | $89 | $199 | $299 | Custom |
| **Price (yearly)** | Free | $90 | $190 | $890 | $1,990 | $2,990 | Custom |
| **Yearly savings** | — | $18 | $38 | $178 | $398 | $598 | — |
| **Seats** | 1 | 1 | 1 | 1–5 | 6–15 | 16–50 | 51+ |
| **Trees** | 5 | 50 | Unlimited | Unlimited | Unlimited | Unlimited | Unlimited |
| **AI Wizard (messages/mo)** | 20 | 50 | Unlimited | Unlimited | Unlimited | Unlimited | Unlimited |
| **Identifying Wand (uses/mo)** | 10 | 50 | Unlimited | Unlimited | Unlimited | Unlimited | Unlimited |
| **Data/map export** | Limited | Limited | Unlimited | Unlimited | Unlimited | Unlimited | Unlimited |
| **GZ-TIN access** | Limited | Limited | Unlimited | Unlimited | Unlimited | Unlimited | Unlimited |
| **Properties** | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Clients** | ✗ | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Workflow** (requests/quotes/jobs/invoices) | ✗ | ✗ | ✗ | Full | Full | Full | Full |
| **Team features** | ✗ | ✗ | ✗ | ✓ | ✓ | ✓ | ✓ |
| **Widgets** | ✗ | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Support** | Basic | Basic | Priority | Priority | Priority | Priority | Dedicated CS (Enterprise by quote) |

*Marketing stance:* API access, white-label, and SLA are **not** sold as self-serve add-ons on Mid/Large tiers. **Enterprise** is **quote-only** (contact sales); white-label is discussed in that context.

---

## Tier Detail

### Home — Free

| | |
|---|---|
| **Badge** | No credit card required |
| **Subtext** | Forever |
| **Monthly** | Free |
| **Yearly** | Free |
| **Target** | Homeowners mapping their property trees |

**What's included:**
- Groundzy Catalog
- Up to 5 trees
- Up to 20 AI Wizard messages/month
- Up to 10 Identifying Wand uses/month
- GZ-TIN (limited)
- Basic support
- Hire a Pro (marketplace access)

---

### Plus — $9/mo or $90/yr

| | |
|---|---|
| **Badge** | More trees & limits |
| **Subtext** | Enhanced |
| **Monthly** | $9/month |
| **Yearly** | $90/year |
| **Yearly savings** | $18 vs monthly |
| **Target** | Homeowners/enthusiasts who need more capacity |

**What's included (everything in Home, plus):**
- Up to 50 trees (10× more than Home)
- Up to 50 AI Wizard messages/month
- Up to 50 Identifying Wand uses/month
- **Properties** management (`crm.properties` capability)
- Property-focused tools
- Hire a Pro

**What Plus does NOT include vs Pro:**
- No Clients management (clients require Pro+)
- No Widgets
- No data/map export (limited)
- No priority support
- No workflow (requests, quotes, jobs, invoices)

---

### Pro — $19/mo or $190/yr

| | |
|---|---|
| **Badge** | 14-day free trial |
| **Subtext** | Only |
| **Monthly** | $19/month |
| **Yearly** | $190/year |
| **Yearly savings** | $38 vs monthly |
| **Most popular** | Yes |
| **Target** | Professionals, arborists, landscapers, students |

**What's included (everything in Home, plus):**
- Unlimited trees
- Unlimited AI Wizard messages
- Unlimited Identifying Wand uses
- Unlimited markers & GZ-TIN access
- Unlimited data/map export
- **Clients** management (`crm.clients` capability)
- **Properties** management (`crm.properties` capability)
- Widgets
- Priority support

**What Pro does NOT include vs Teams:**
- No team collaboration
- No team dashboards
- No admin controls / user management
- No dedicated account management
- No seasonal seats
- Workflow pipeline (requests, quotes, jobs, invoices) is Teams-only

---

### Teams (Small) — $89/mo or $890/yr

| | |
|---|---|
| **Badge** | 14-day free trial |
| **Subtext** | Starting at |
| **Seats** | 1–5 users |
| **Monthly** | $89/month |
| **Yearly** | $890/year |
| **Yearly savings** | $178 vs monthly |
| **Target** | Growing tree care businesses, property teams |

**What's included (everything in Pro, plus):**
- Full workflow: requests, quotes, jobs, invoices — shared across team
- Team collaboration
- Team dashboards
- Admin controls & user management
- Dedicated account management
- Seasonal seats (add/remove anytime, pro-rated)

---

### Teams (Mid) — $199/mo or $1,990/yr

| | |
|---|---|
| **Seats** | 6–15 users |
| **Monthly** | $199/month |
| **Yearly** | $1,990/year |
| **Yearly savings** | $398 vs monthly |

**What's included:** Same feature set as Small Team with a larger seat band (6–15 users). Mid Team is a **seat/pricing band**, not a separate feature bundle in the product.

---

### Teams (Large) — $299/mo or $2,990/yr

| | |
|---|---|
| **Seats** | 16–50 users |
| **Monthly** | $299/month |
| **Yearly** | $2,990/year |
| **Yearly savings** | $598 vs monthly |

**What's included:** Same feature set as other team tiers with a larger seat band (16–50 users).

---

### Enterprise — Custom (quote only)

| | |
|---|---|
| **Seats** | 51+ |
| **Pricing** | Custom quote (no self-serve Stripe checkout in auth) |
| **Contact** | support@groundzy.com |

**Positioning:** Sold **by quote** through sales. **White-labeling** and **dedicated customer success** are discussed as part of the Enterprise conversation—not as checkout add-ons on Mid/Large self-serve tiers.

---

## Billing Rules

- **No contracts** — cancel anytime
- **Yearly = ~10× monthly** (approximately 2 months free)
- **Prices in USD**
- **Seasonal seats**: pro-rated, add/remove anytime, price shown at checkout
- **Usage resets**: monthly, on signup-day anniversary (or calendar month if signup date unknown)
- **Active subscription statuses**: `active` and `trialing` grant paid-tier access; all others fall back to Home

### Free trial policy

- **Pro and Teams** start with a 14-day free trial when purchased through Stripe Checkout.
- A valid payment method is required to start a Pro or Teams trial.
- Home remains free with no card required.
- Plus does not receive a free trial.
- Trial behavior is applied in code at subscription creation using the existing Price IDs; no separate trial Prices are required.
- During the trial, `subscription.status = "trialing"` grants paid-tier access.
- Unless canceled before trial end, Stripe converts the subscription to the selected monthly or yearly paid plan.

---

## Runtime Tier Names (Firestore)

Stored in `users/{uid}.subscription.tier`. Values in Firestore use these exact strings:

| Checkout plan | Firestore `tier` value(s) |
|---|---|
| `home` | `"Home"` |
| `plus` | `"Plus"` |
| `pro` | `"Pro"` |
| `teams` → small | `"Small Team"` |
| `teams` → mid | `"Mid Team"` |
| `teams` → large | `"Large Team"` |
| `enterprise` (onboarding selection; no Stripe price) | `"Enterprise"` (incomplete until contracted) |
| _(custom / sales)_ | `"Enterprise"` |

**Sources:** `auth.groundzy/lib/billing/tier-model.ts`, `app.groundzy/lib/billing/tier-model.ts`

---

## Capability System (app.groundzy)

Defined in `app.groundzy/lib/capabilities.ts`. Hook: `useCapabilities()`.

| Capability key | Home | Plus | Pro | Teams |
|---|---|---|---|---|
| `workitems.read_tree` | ✓ | ✓ | ✓ | ✓ |
| `workitems.write_tree` | ✓ | ✓ | ✓ | ✓ |
| `legacy.pro_plus_teams_surfaces` | ✗ | ✓ | ✓ | ✓ |
| `crm.properties` | ✗ | ✓ | ✓ | ✓ |
| `crm.clients` | ✗ | ✗ | ✓ | ✓ |
| `workflow.pipeline` | ✗ | ✗ | ✗ | ✓ |
| `workitems.read_org` | ✗ | ✗ | ✗ | ✓ |
| `map.layers_jobs` | ✗ | ✗ | ✗ | ✓ |

> Note: `workflow.pipeline` is the team/org pipeline; Pro uses workflow in an individual-practice scope. Access control distinguishes individual vs team org context.

---

## Usage Limits (app.groundzy)

Defined in `app.groundzy/lib/ai-usage.ts`. Enforced server-side.

| Limit type | Home | Plus | Pro | All Teams |
|---|---|---|---|---|
| **Trees** | 5 | 50 | Unlimited | Unlimited |
| **AI Wizard messages / month** | 20 | 50 | Unlimited | Unlimited |
| **Identifying Wand uses / month** | 10 | 50 | Unlimited | Unlimited |

**API endpoints:**
- `GET /api/trees/usage` → `{ trees: { used, limit, isUnlimited } }`
- `GET /api/ai/usage` → `{ chat: { used, limit }, plantnet: { used, limit, isUnlimited } }`
- `POST /api/ai/chat` → `429 Too Many Requests` when monthly limit reached
- `POST /api/species/identify/reserve` → checks monthly PlantNet quota

---

## Drawer Visibility by Tier (app.groundzy)

Defined in `app.groundzy/lib/drawers.ts`.

| Drawer | Home | Plus | Pro | Teams |
|---|---|---|---|---|
| Dashboard | ✓ | ✓ | ✓ | ✓ |
| Map / Trees | ✓ | ✓ | ✓ | ✓ |
| Clients & Properties | ✗ | ✓ | ✓ | ✓ |
| Requests | ✗ | ✗ | ✓ | ✓ |
| Quotes | ✗ | ✗ | ✓ | ✓ |
| Jobs | ✗ | ✗ | ✓ | ✓ |
| Invoices | ✗ | ✗ | ✓ | ✓ |
| Hire a Pro | ✓ | ✓ | ✓ | ✓ |
| Upgrade to Plus | ✓ | ✗ | ✗ | ✗ |
| Upgrade to Pro | ✓ | ✓ | ✗ | ✗ |
| Upgrade to Teams | ✓ | ✓ | ✓ | ✗ |

---

## Upgrade Prompts & Paywalls (app.groundzy)

### Upgrade to Plus drawer
**File:** `app/drawers/upgrade-to-plus.tsx`  
**Visible to:** Home only  
**Heading:** "More trees. Smarter AI."  
**CTA:** "Upgrade to Plus"  
**Features promoted:**
- More trees (50 vs 5)
- More AI Wizard messages
- More Identifying Wand uses
- Hire a Pro

### Upgrade to Pro drawer
**File:** `app/drawers/upgrade-to-pro.tsx`  
**Visible to:** Home, Plus  
**Heading:** "Unlimited trees. Unlimited AI."  
**CTA:** "Upgrade to Pro"  
**Features promoted:**
- Unlimited trees
- Unlimited AI Wizard
- Unlimited Identifying Wand
- Clients & properties management
- Weather integration
- Priority support

### Upgrade to Teams drawer
**File:** `app/drawers/upgrade-to-teams.tsx`  
**Visible to:** Home, Plus, Pro  
**Heading:** "Workflow for your team."  
**CTA:** "Continue to checkout"  
**Features promoted:**
- Full workflow: requests, quotes, jobs, invoices
- Team collaboration & dashboards
- Admin controls

### Inline upgrade CTAs

**`UpgradeLimitCta` component** (`components/upgrade-limit-cta.tsx`)  
Shown when user hits a usage limit. Buttons: "Upgrade to Plus" / "Upgrade to Pro".

**`WorkflowViewAccessDenied`** (`app/drawers/components/WorkflowViewAccessDenied.tsx`)  
Shown when a user without `workflow.pipeline` capability tries to open a workflow record.  
CTA: "Upgrade to Teams"

### Access control hooks

| Hook | File | Gate |
|---|---|---|
| `useCapabilities()` | `hooks/useCapabilities.ts` | All capability checks |
| `useProOrTeamsAccess()` | `hooks/useProOrTeamsAccess.ts` | `legacy.pro_plus_teams_surfaces` |
| `useTeamsOnlyAccess()` | `hooks/useTeamsOnlyAccess.ts` | `workflow.pipeline` (teams only) |
| `useGlobalEntitlements()` | `hooks/useGlobalEntitlements.ts` | Workflow list surface (Pro or Teams) |
| `useIsPlusOnlyProperties()` | `hooks/useProOrTeamsAccess.ts` | Plus with no client access |

---

## Pricing Tables by Surface

### auth.groundzy (Onboarding)

**Component:** `components/custom-pricing-table.tsx`  
**Tier order displayed:** Home → Plus → Pro → Teams → Enterprise (Enterprise is quote-only: contact CTA, no Stripe checkout)  
**Team sizes shown:** Small (1–5), Mid (6–15), Large (16–50) — team size selector appears only when Teams is selected  
**Billing toggle:** Monthly / Yearly  
**Locales:** English (en), Spanish (es), French (fr)  
**Feature source:** `lib/pricing.ts` → `PRICING_PLANS` + `PRICING_FEATURE_KEYS`  
**i18n source:** `lib/i18n/messages.ts` → `pricing.*`

Key i18n keys:
```
pricing.choosePlan
pricing.noContracts
pricing.monthly / pricing.yearly
pricing.mostPopular
pricing.recommendedForYou
pricing.saveYearlyVsMonthly → "Save {amount}/year vs paying monthly"
pricing.seatRange → "{min}–{max} users"
pricing.tiers.{tier}.name / .subtext / .badge / .description
pricing.tiers.{tier}.features.{key}
pricing.tiers.teams.smallTeam / .midTeam / .largeTeam
```

### groundzy.com (Marketing)

**File:** `index.html` lines 584–792 (pricing section + FAQ)  
**Tier order displayed:** Home → Pro → Teams (Plus is shown as an optional add-on)  
**Notable differences vs auth.groundzy:**
- Plus remains **Home-centric** on the marketing site (optional add-on copy); auth lists Plus as a first-class plan.
- Pro FAQ uses **workflow** (not “solo workflow”) for requests through invoices.
- Team tier FAQ does **not** promise API, premium support, white-label, or SLA as tier add-ons.
- Enterprise: custom quote by email (`support@groundzy.com`); white-label framed under Enterprise.
- Identifying Wand / coverage: public copy references **broad taxonomic coverage (78,000+ taxa)** for the Wand; **Groundzy Catalog** references **1,000+** species where catalog breadth is described.
- GZ-TIN highlighted as "Included in Every Plan"

---

## Discrepancies Between Surfaces

| Topic | auth.groundzy | groundzy.com | app.groundzy |
|---|---|---|---|
| Plus presentation | First-class tier with own card | Optional add-on to Home (Home-centric positioning) | First-class tier |
| Pro workflow description | Feature list + workflow wording | "Workflow (requests through invoices…)" in FAQ | Capability + policy |
| Teams description | i18n: growing crews / municipalities / shared inventories | FAQ aligned; no API/SLA/wl promises as tier add-ons | — |
| Enterprise | Fifth column; quote-only flow (mailto + app) | FAQ: quote + white-label / dedicated CS | `RUNTIME_TIERS` includes Enterprise |
| Identifying Wand / catalog (marketing) | Unlimited vs limited (usage) | 78,000+ taxa (Wand); 1,000+ catalog where relevant | Numeric limits in `ai-usage.ts` |
| Plus price | $9/mo or $90/yr | $9/mo or $90/yr (shown as add-on) | Same |

---

## Source Files Quick Reference

| File | Repo | Contents |
|---|---|---|
| `lib/pricing.ts` | auth.groundzy | `PRICING_PLANS`, `PRICING_FEATURE_KEYS`, `TEAM_SIZES`, `yearlySavingsDollars()` |
| `lib/billing/tier-model.ts` | auth.groundzy | `RuntimeTier` definitions |
| `lib/tier-utils.ts` | auth.groundzy | `getEffectiveSubscriptionTier()`, `isTierInGroup()`, `hasPaidTierIncomplete()` |
| `lib/i18n/messages.ts` | auth.groundzy | All display text for EN / ES / FR |
| `components/custom-pricing-table.tsx` | auth.groundzy | Pricing table UI component |
| `lib/onboarding-tier-suggestion.ts` | auth.groundzy | `suggestTier()` — weighted scoring from onboarding answers |
| `index.html` | groundzy.com | Marketing pricing section (lines 584–792) |
| `lib/billing/tier-model.ts` | app.groundzy | Same `RuntimeTier` definitions |
| `lib/billing/stripe-tier-map.ts` | app.groundzy | Stripe price ID ↔ tier mapping |
| `lib/ai-usage.ts` | app.groundzy | `TREE_LIMITS`, `AI_USAGE_LIMITS`, `PLANTNET_USAGE_LIMITS` |
| `lib/capabilities.ts` | app.groundzy | `getCapabilitiesForTier()` |
| `lib/drawers.ts` | app.groundzy | Drawer registry with `visibleForTiers` |
| `app/drawers/upgrade-to-plus.tsx` | app.groundzy | Upgrade to Plus prompt |
| `app/drawers/upgrade-to-pro.tsx` | app.groundzy | Upgrade to Pro prompt |
| `app/drawers/upgrade-to-teams.tsx` | app.groundzy | Upgrade to Teams prompt |
| `components/upgrade-limit-cta.tsx` | app.groundzy | Inline usage limit CTA |
