# Stripe webhooks (operations)

## Canonical configuration

Groundzy uses **two** Next.js apps that each expose a webhook route:

| App | Route |
|-----|--------|
| Auth | `https://auth.groundzy.com/api/stripe/webhook` |
| Main app | `https://app.groundzy.com/api/stripe/webhook` |

**Both write to the same Firestore `users` collection.** Stripe should send each event to **only one** endpoint in production, or you risk duplicate processing and conflicting updates.

### Required action (Stripe Dashboard)

1. Open **Developers → Webhooks**.
2. Ensure **one** endpoint receives at least: `customer.subscription.created`, `customer.subscription.updated`, `customer.subscription.deleted`.
3. Prefer the **auth** endpoint if onboarding checkout is the primary path, or the **app** endpoint if most subscriptions originate in-app—**pick one** and disable or narrow the other so the same event is not delivered twice.

### Secrets

Each deployed app needs `STRIPE_WEBHOOK_SECRET` matching the signing secret of the endpoint that targets that app.
