# Pricing

**Sources:** `types/signup-flow.ts` (`PRICING_TIERS`), `lib/stripe-config.ts` (Stripe price IDs), `docs/features/README.md` (feature matrix). **Live commerce** may differ—Stripe Dashboard is authoritative for production.

---

## Catalog (signup / marketing structs)

| Plan | Monthly | Yearly | Notes |
|------|---------|--------|--------|
| **Home** | $0 | $0 | Free |
| **Pro** | $19 | $171 | Individual paid |
| **Teams** (entry band in struct) | $89 | $801 | Marketing rollup; team-sized bands below |

**Team size bands** (`PRICING_TIERS.teams.teamSizes`):

| Band | Monthly | Yearly | Seasonal seat hint (struct) |
|------|---------|--------|-------------------------------|
| Small (10) | $89 | $801 | 14 |
| Mid (25) | $199 | $1,791 | 8 |
| Large (50) | $299 | $2,691 | 6 |
| Enterprise | *null* | *null* | *null* — custom / sales |

**Plus:** Not in `PRICING_TIERS` object; **Stripe** config treats **Plus** as **yearly-only** (`getPriceId` returns null for monthly). Comment in `stripe-config.ts` references **~$9.99/year** for Plus product; price ID via `STRIPE_PRICE_PLUS_YEARLY` env or default id.

---

## Stripe mapping

- **`getPriceId(tier, billingCycle, teamSize)`** maps Pro, Plus (yearly only), and Teams sizes → **Stripe price IDs** (hardcoded in `lib/stripe-config.ts`; rotate for new Stripe products).
- **`getTierFromPriceId`** reverses mapping for webhooks when metadata is stale.
- **Enterprise** pricing in signup struct is **null**—checkout likely custom or off-app.

---

## Billing cycles

- **Monthly** and **yearly** for Pro and team tiers in `getPriceId`.
- **Plus:** yearly only in code path.

---

## Feature bundles (marketing copy in `PRICING_TIERS`)

- **Home:** limited trees, limited wand/wizard, basic support.
- **Pro:** unlimited markers/GZ-TIN, unlimited wand/wizard, export, clients & properties, priority support, widgets.
- **Teams:** Pro plus full workflow, collaboration, dashboards, admin, account management, seasonal seats.

**Overlap with docs:** `docs/features/README.md` tier table (Home 5 trees, etc.) may differ from bullet list above—**treat README as feature-doc snapshot** and `PRICING_TIERS.features` as marketing strings; verify in-app limits for ground truth.

---

## Risks & constraints

| Risk | Detail |
|------|--------|
| **Dual sources** | TypeScript catalog vs Stripe Dashboard vs in-app limits |
| **Price ID churn** | Hardcoded ids in repo require deploys when Stripe prices change |
| **Plus omission** | `PlanTier` type is `home \| pro \| teams` only—**Plus** exists only in runtime `SubscriptionTier` and Stripe |

---

## Related

- [`tier-rules.md`](./tier-rules.md)
- `Groundzy v3/01-product/tier-system.md`
