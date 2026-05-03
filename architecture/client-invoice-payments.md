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
| Guest pay settlement (after Elements confirms) | `POST /api/client-hub/invoice-pay/confirm` — idempotent; use with Connect webhooks |
| Staff PaymentIntent / saved card | `POST /api/stripe/connect/create-payment-intent`, `POST /api/stripe/connect/charge-saved` |
| List saved cards + default PM | `GET /api/stripe/connect/payment-methods` |
| Add / remove / default card (API) | `POST /api/stripe/connect/setup-payment-method`, `detach-payment-method`, `set-default-payment-method` |
| CRM UI | `ClientPaymentMethodsSection` in **view-client** drawer (Pro/Teams, `org.client.update` + Stripe Connect ready) |
| Refund | `POST /api/stripe/connect/refund` |
| Recurring (treatment subscription) | `POST /api/stripe/connect/recurring-subscription` |
| Recurring (invoicing / schedule, Stripe invoices per cycle) | `POST /api/stripe/connect/recurring-invoice-schedule` |
| Webhooks | **`POST /api/stripe/connect-webhook`** — `payment_intent.*`, `account.updated`, `charge.refunded`, `invoice.paid` (recurring contract mirror) |

## Stripe Dashboard — Connect webhook

Client invoice card charges use **direct charges** on the **connected account** (`PaymentIntents.create` … `{ stripeAccount: connectedAccountId }`). Stripe sends `payment_intent.succeeded` (and related events) **for that connected account**. Your app must register a **second** webhook endpoint on the **app** host that listens to those events—not only events on the platform account.

### Setup checklist

1. **Dashboard:** [Developers → Webhooks](https://dashboard.stripe.com/webhooks) (use **Test** and **Live** mode each as needed).
2. **Add endpoint** URL: `https://<your-app-host>/api/stripe/connect-webhook`  
   When creating the endpoint, enable listening to events on **Connected accounts** (wording may be “Connect” or “Receive events from connected accounts,” depending on the Dashboard version).
3. **Events** — at minimum subscribe to:
   - `payment_intent.succeeded` (invoice settlement)
   - `account.updated` (Connect account status)
   - `charge.refunded` (invoice partial reversal)
   - `invoice.paid` (recurring contract mirror, if used)
4. **Signing secret:** Copy the endpoint’s signing secret into **`STRIPE_CONNECT_WEBHOOK_SECRET`** on the **app** deployment (Firebase App Hosting / hosting env). This must **not** be the same secret as subscription webhooks on **auth** (`STRIPE_WEBHOOK_SECRET`).
5. **Mode alignment:** `STRIPE_SECRET_KEY`, `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`, payments, and this webhook must all be **test** or all **live** together.

### Troubleshooting

- **Invoice stays “Awaiting payment” after the client paid:** In the Dashboard, open the PaymentIntent → **Events**. Confirm `payment_intent.succeeded` was **delivered** to `.../api/stripe/connect-webhook` with a **2xx** response. If there is no delivery, the endpoint is likely registered only for the **platform** account—add or fix a **Connect** listener.
- **HTTP 400 on the webhook:** Usually a **wrong signing secret** (copy the secret for this endpoint only).
- **HTTP 503 “Connect webhook not configured”:** `STRIPE_CONNECT_WEBHOOK_SECRET` or `STRIPE_SECRET_KEY` is missing on the app server.
- **Settlement still delayed:** The guest pay UI also calls **`POST /api/client-hub/invoice-pay/confirm`** after `confirmPayment` so Firestore can update when the browser session completes, even if the webhook is slow or misconfigured. Fix the webhook for refunds, disputes, and server-to-server reliability.

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
