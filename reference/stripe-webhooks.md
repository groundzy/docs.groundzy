# Stripe webhooks (operations)

## Canonical configuration

Groundzy has historically had **two** Next.js apps that expose a webhook route:

| App | Route |
|-----|--------|
| Auth | `https://auth.groundzy.com/api/stripe/webhook` |
| Main app | `https://app.groundzy.com/api/stripe/webhook` (deprecated; returns 410 in current app code) |

Production Stripe events must target **auth only**. The app webhook is deprecated to avoid duplicate processing and conflicting updates.

### Required action (Stripe Dashboard)

1. Open **Developers → Webhooks**.
2. Ensure the auth endpoint receives at least: `customer.subscription.created`, `customer.subscription.updated`, `customer.subscription.deleted`.
3. Remove or disable the app endpoint in production.
4. For Pro and Teams trials, confirm `customer.subscription.created` / `updated` events mirror `trialing`, `trial_start`, and `trial_end` to Firestore.

### Secrets

Each deployed app needs `STRIPE_WEBHOOK_SECRET` matching the signing secret of the endpoint that targets that app.
