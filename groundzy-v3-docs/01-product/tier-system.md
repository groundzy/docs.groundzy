# Tier System

**Authoritative product reference for plans and access.** Canonical entity definitions: [`core-concepts.md`](./core-concepts.md). Do not duplicate tier tables in other docs—**link here**.

---

## How to read this document

- **Runtime / UI access** uses a set of **subscription tier labels** in code (`SubscriptionTier` in `lib/drawer-registry.ts`).
- **Signup / pricing tables** use a different grouping in `types/signup-flow.ts` (**Home**, **Pro**, **Teams** with team sizes)—**Plus** exists in the app but not in that `PlanTier` union.
- **Feature matrix** in `docs/features/README.md` describes Home, Plus, Pro, Teams capabilities at a glance.

v3 should **converge** naming; until then, this doc is the reconciliation layer.

---

## Naming inconsistencies (explicit)

| Area | What appears | Notes |
|------|----------------|--------|
| Signup `PlanTier` | `home` \| `pro` \| `teams` only | **Plus** is not in this type; Plus is still a real runtime tier. |
| Runtime `SubscriptionTier` | `Home`, `Plus`, `Pro`, `Small Team`, `Mid Team`, `Large Team`, `Enterprise` | Granular team sizes; **Teams** in marketing maps to these. |
| Features README | “Plus: Home + unlimited trees, Hire Pro” | Differs from pricing tier feature bullets in `PRICING_TIERS` for Home (which lists limited trees)—use README for **feature bundling**, signup for **commerce**. |
| Paid access | Stripe `subscription.status` | **Pro** and team tiers only grant paid UI when status is **active** or **trialing** (`lib/utils/tier-utils.ts`). |

When docs disagree, **engineering behavior** is defined by `lib/utils/tier-utils.ts`, `lib/drawers.ts`, and Firestore rules—not by marketing tables alone.

---

## Tier summaries (product behavior)

### Home

- **Price:** Free (`PRICING_TIERS` in `types/signup-flow.ts`).
- **Audience:** Property stewards mapping their own trees (`TIER_DESCRIPTIONS`).
- **Core access:** Map, trees with **limits**, AI Wizard / Identifying Wand with limits, weather, dashboard, profile.
- **Hire a Pro:** Available (`docs/features/hire-pro.md`).
- **CRM / Workflow:** Not in tier matrix for full CRM or pipeline.

### Plus

- **Runtime tier** in `SubscriptionTier`; **yearly** Stripe path noted in `lib/stripe-config.ts` (Plus is yearly-only in that module).
- **Positioning:** Higher limits than Home for trees and AI usage (exact multipliers in product copy / i18n, not repeated here to avoid drift).
- **Hire a Pro:** Included in feature messaging.
- **CRM / Workflow:** Does **not** unlock Pro CRM drawers in the product structure.

### Pro

- **Individual paid** plan in pricing metadata: expanded limits, export, **clients & properties**, widgets, priority support (`PRICING_TIERS.pro.features`).
- **CRM:** Full **clients/properties** CRM per feature matrix.
- **Workflow:** **No** full **requests → quotes → jobs → invoices** drawer set in `docs/features/README.md` (“Pro | Plus + Clients, Properties, full CRM (no workflow)”).
- **Hire a Pro:** Hidden for this persona (service provider stance).

### Teams (Small / Mid / Large / Enterprise)

- **Marketing:** “Teams” with **team size** bands and pricing (`PRICING_TIERS.teams.teamSizes`).
- **Runtime:** Separate labels **Small Team**, **Mid Team**, **Large Team**, **Enterprise** (`SubscriptionTier`).
- **Adds on Pro:** Collaboration, dashboards, admin, seasonal seats, **full Workflow** (requests, quotes, jobs, invoices), dedicated account management per pricing copy.

---

## Feature matrix (from `docs/features/README.md`)

| Tier | Features (summary) |
|------|---------------------|
| **Home** | Dashboard, Trees (5 limit in this table), Weather, AI (Identify, Chat), Map, Profile, Hire Pro |
| **Plus** | Home + unlimited trees (per this table), Hire Pro |
| **Pro** | Plus + Clients, Properties, full CRM **(no workflow)** |
| **Teams** | Pro + Requests, Quotes, Jobs, Invoices, Team Settings |

**Conflict handling:** Tree count limits for Home/Plus differ between **README matrix** and **signup feature bullets**—treat README as the **feature-doc snapshot** and pricing/signup as **commerce**; verify in app for current limits.

---

## Operational rules (paid tiers)

- **Effective tier** for gating may fall back to **Home** if subscription is not active (`getEffectiveSubscriptionTier` in `lib/utils/tier-utils.ts`).
- **Drawer visibility** is tier-gated in `lib/drawers.ts` / registry metadata.

---

## Tier checks (v3 — single pattern)

**Rule:** No feature should invent **ad hoc** tier checks in multiple shapes (drawer-only + random util + duplicate API guard) without going through the **same** documented path.

**Intended mechanisms (today):**

- **Effective tier:** `lib/utils/tier-utils.ts` (`getEffectiveSubscriptionTier`, etc.).
- **Drawer visibility:** `visibleForTiers` on drawer metadata (`lib/drawers.ts`, `lib/drawer-registry.ts`).
- **Feature hooks:** `useProOrTeamsAccess`, `useTeamsOnlyAccess`, etc.—use **consistently** for the same capability; prefer consolidating over time.

**Not security:** Tier never replaces Firestore rules ([`../05-data/permissions.md`](../05-data/permissions.md) execution order).

When adding a new gated surface, **extend the central helpers or registry**, document it here or in [`../09-business/tier-rules.md`](../09-business/tier-rules.md), and avoid one-off string comparisons to `"Pro"` scattered in components.

---

## Groundzy v3 direction

- **One tier story** in UX and schema: reconcile **Plus**, **PlanTier**, and **SubscriptionTier** naming.
- **One place** documents limits (trees, AI, etc.) to avoid README vs signup drift.

---

## Related

- [`core-concepts.md`](./core-concepts.md)
- [`personas.md`](./personas.md)
- [`user-journeys.md`](./user-journeys.md)
