# Backfill: subscription tier / Plus integrity

## When to use

Users who **paid for Groundzy Plus** while Firestore still had `subscription.tier: "Home"` (legacy auth checkout metadata bug) need a one-time correction. Prefer fixing via **Stripe** as source of truth.

## Recommended approach

1. In **Stripe → Customers**, find the subscription.
2. Confirm the **Price** maps to Plus (product `home_plus` / Plus yearly or monthly).
3. In **Firestore** `users/{uid}`, set:
   - `subscription.tier` = `Plus`
   - `subscription.status` = `active` (or `trialing` if applicable)
4. Trigger entitlement recompute: call the same logic as the app webhook by **updating the subscription** in Stripe (e.g. toggle metadata) or run a small admin script that loads the user doc and calls `syncEntitlementsForUser` (app) after correcting the tier.

## Manual QA (after deploy)

- [ ] Auth onboarding: Home → Home Plus → Stripe Checkout → Firestore `subscription.tier` is **Plus** after webhook.
- [ ] App profile: Plus yearly checkout still works.
- [ ] App profile: Pro checkout still works.
- [ ] Teams: checkout → team created; `subscription.tier` is **Small Team** / **Mid Team** / **Large Team** (not ambiguous `Teams` only).
- [ ] Stripe **Customer portal** plan change updates tier via **price ID** mapping.
- [ ] Only **one** webhook endpoint is configured in Stripe for production (see [stripe-webhooks.md](../reference/stripe-webhooks.md)).
