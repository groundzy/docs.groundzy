# Billing & subscriptions

## What it does

**Stripe** checkout, customer portal, subscription create/update, webhooks updating **user** and **team** documents. Tier drives **drawer visibility** and **effective access** (`getEffectiveSubscriptionTier`).

## Who uses it

Paid **Pro** and **Teams** users; **Plus** yearly path in Stripe config (`lib/stripe-config.ts`). **Home** is free.

## Data involved

- **User doc:** `subscription` object (tier, status, Stripe ids)—see `lib/utils/tier-utils.ts`.
- **Team doc:** `subscriptionStatus`, `subscriptionId`, billing fields (`types/team.ts`).
- **API:** `app/api/stripe/*` (checkout, portal, webhook, payment retry, etc.).
- **No** separate `subscriptions` collection in the feature overview—state on **user/team**.

## UI patterns

Upgrade drawers (`upgrade-to-plus`, `upgrade-to-pro`, `upgrade-to-teams`), checkout success routes, profile billing entry points.

## Dependencies

- Stripe keys (server-only secrets)
- Firebase Auth for user id
- Webhook signature verification on `/api/stripe/webhook`

## Inconsistencies & overlaps

- **Tier naming:** `PlanTier` in signup vs `SubscriptionTier` in registry vs marketing—see `Groundzy v3/01-product/tier-system.md`.
- **Effective tier** falls back to **Home** if subscription inactive—UI must match **Stripe truth** or users see confusing partial access.
