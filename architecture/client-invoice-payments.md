# Client invoice payments (Stripe Connect)

This document describes how Groundzy processes **client-facing** card payments (invoices, saved cards, recurring treatment subscriptions) using **Stripe Connect Express** on the **app** deployment. It is separate from **Groundzy SaaS subscription** billing, which uses Stripe Checkout and webhooks on **auth.groundzy**.

## Product scope

- **Eligible tiers:** Pro and Teams runtime tiers only. Home and Plus do not have workflow or client payment surfaces.
- **Platform fee:** Optional basis points via `GROUNDZY_CONNECT_PLATFORM_FEE_BPS` or per-team override in `teams/{id}.clientPaymentSettings.platformFeeBpsOverride`.
- **Guest pay:** End clients pay without a Groundzy login via `/pay/{portalToken}` and `POST /api/client-hub/invoice-pay`.

## Architecture

| Concern | Location |
|--------|----------|
| Connect account create / onboarding | `POST /api/stripe/connect/create-account`, `POST /api/stripe/connect/account-link` |
| Team payment prefs (fee payer) | `GET/POST /api/stripe/connect/client-payment-settings` |
| Guest PaymentIntent | `POST /api/client-hub/invoice-pay` |
| Staff PaymentIntent / saved card | `POST /api/stripe/connect/create-payment-intent`, `POST /api/stripe/connect/charge-saved` |
| List saved cards + default PM | `GET /api/stripe/connect/payment-methods` |
| Add / remove / default card (API) | `POST /api/stripe/connect/setup-payment-method`, `detach-payment-method`, `set-default-payment-method` |
| CRM UI | `ClientPaymentMethodsSection` in **view-client** drawer (Pro/Teams, `org.client.update` + Stripe Connect ready) |
| Refund | `POST /api/stripe/connect/refund` |
| Recurring (treatment subscription) | `POST /api/stripe/connect/recurring-subscription` |
| Recurring (invoicing / schedule, Stripe invoices per cycle) | `POST /api/stripe/connect/recurring-invoice-schedule` |
| Webhooks | **`POST /api/stripe/connect-webhook`** — `payment_intent.*`, `account.updated`, `charge.refunded`, `invoice.paid` (recurring contract mirror) |

## Environment variables (app)

| Variable | Purpose |
|----------|---------|
| `STRIPE_SECRET_KEY` | Platform secret (shared with subscription flows) |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Elements / `loadStripe` |
| `STRIPE_CONNECT_WEBHOOK_SECRET` | Signing secret for **Connect** webhook endpoint only |
| `GROUNDZY_CONNECT_PLATFORM_FEE_BPS` | Default platform fee (0–10000), optional |
| `GROUNDZY_CONNECT_STRIPE_PASSTHROUGH_BPS` | When fee payer is `client`, rough uplift on charge amount (default 350 bps) |

## Firestore

- `teams.{organizationId}.stripeConnect` — Express `accountId`, `chargesEnabled`, etc. (written by API + `account.updated` webhook).
- `teams.{organizationId}.clientPaymentSettings` — `stripeFeePayer`, optional `platformFeeBpsOverride`.
- `clients.{clientId}.stripeCustomerId` — Customer on the **connected** account (server-only client writes).
- `billing_recurring_contracts` — Groundzy mirror for **subscription** (treatment) and **invoicing** schedule modes; `invoice.paid` updates `lastPaidAt` / `lastStripeInvoiceId` when contract metadata matches.
- Clients cannot mutate `stripeConnect`, `clientPaymentSettings`, `stripeCustomerId`, or invoice `lastStripePaymentIntentId` via security rules; settlement is webhook-authoritative.

## Org actions

- `org.payments.connect_onboard` — create Express account and onboarding links (owner/admin).
- `org.payments.charge_card` — charge saved card / staff flows (matrix + sales preset denies).
- `org.payments.refund` — refunds API.

See `lib/groundzy/policy/org-action.ts` and `preset-org-action.ts` for the matrix; update `groundzy-v3-docs/05-data/org-action-policy-matrix.md` when changing actions.
