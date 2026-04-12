# Growth strategy (product mechanics)

This document describes **observable monetization and expansion mechanics** in the repository—not a corporate strategy. For revenue goals, use internal business planning outside this repo.

---

## 1. Freemium entry

- **Home** at **$0** (`PRICING_TIERS`) lowers barrier; **limits** on trees and AI (see feature README vs `PRICING_TIERS`—verify in-app) encourage upgrades.

---

## 2. Upgrade ladder (product)

| Step | Mechanism in repo |
|------|-------------------|
| **Home → Plus** | Yearly Stripe **Plus** price (`getPriceId`, `STRIPE_PRICE_PLUS_YEARLY`); higher limits + Hire a Pro in messaging |
| **Plus → Pro** | Pro checkout / Stripe |
| **Pro → Teams** | Team tiers with **Small / Mid / Large** price bands; `invite_codes`, team signup |

**Upgrade drawers:** `upgrade-to-plus`, `upgrade-to-pro`, `upgrade-to-teams` (`lib/drawers.ts`).

---

## 3. Expansion within organizations

- **Team size bands** with **different monthly/yearly prices** and **seasonal seat** hints in `PRICING_TIERS.teams.teamSizes`.
- **Enterprise** row with **null** prices — signals **sales-assisted** or custom deal path (not self-serve in struct).

---

## 4. Cross-sell surfaces

| Surface | Role |
|---------|------|
| **Dashboard / limits** | `useTreeUsage`, `useAiUsage` — upgrade CTAs |
| **Profile** | Subscription summary, Stripe portal link |
| **Hire a Pro** | Connects homeowners to pros (ecosystem, not direct app subscription revenue) |

---

## 5. Retention & billing operations

- **Stripe Customer Portal** — payment method, cancel (routes under `app/api/stripe/`).
- **Webhooks** — subscription state sync to Firestore; **notifications** may fire on events (`lib/stripe-webhook-handlers.ts`).

---

## 6. Risks to clean alignment (revenue + product)

| Issue | Effect |
|-------|--------|
| **Tier name drift** | Analytics and support tickets confuse Home/Plus/Pro/Teams vs signup `PlanTier` |
| **Effective tier vs raw tier** | User sees “Pro” in DB but gets Home UX if status inactive — support load |
| **Feature matrix vs marketing bullets** | Tree limits and AI limits may not match `PRICING_TIERS` strings—**trust the app** for enforcement |

---

## 7. v3 direction

- **Single revenue story** in docs + schema: tier names, prices, and entitlements **one table** maintained with Stripe.
- **Capability flags** derived from subscription + org, not scattered hooks—**cleaner upsell** and reporting.

---

## Related

- [`pricing.md`](./pricing.md)
- [`tier-rules.md`](./tier-rules.md)
- `Groundzy v3/08-integrations/stripe.md`
