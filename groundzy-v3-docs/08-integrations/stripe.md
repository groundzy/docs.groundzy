# Stripe

## What it does

**Payments and subscriptions:** Checkout sessions, customer portal, subscription lifecycle, payment method updates, webhooks that update **Firestore** user/team subscription fields. Price IDs map to tiers (`lib/stripe-config.ts`).

## How it’s used

| Area | Connection |
|------|------------|
| **Client** | `@stripe/stripe-js`, `@stripe/react-stripe-js` — publishable key only |
| **Server** | `app/api/stripe/*` — `create-checkout-session`, `create-portal-session`, `webhook`, `create-customer`, `create-subscription`, retries, beta flows, etc. |
| **Webhooks** | `lib/stripe-webhook-handlers.ts` — updates Firestore, may create **`notifications`** docs |

**Env:** `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`; optional price overrides (`docs/reference/environment-variables.md`).

## Risks & constraints

| Risk | Mitigation / note |
|------|-------------------|
| **Secret key exposure** | Never bundle in client; only server routes / actions |
| **Webhook authenticity** | Verify signature with `STRIPE_WEBHOOK_SECRET` |
| **Idempotency / duplicate events** | Handlers should tolerate Stripe retries (implementation detail per route) |
| **Price ID drift** | `getPriceId` maps tiers to Stripe; product changes require config updates |
| **Test vs live** | Keys are environment-specific; wrong key = wrong data or failed checkout |

## Related

- `lib/utils/tier-utils.ts` — effective tier after subscription state
- `Groundzy v3/06-features/billing.md`
