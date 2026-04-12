# Stripe Integration

Stripe payment integration.

## Config

- **lib/stripe-config.ts** – Price IDs, config
- **lib/stripe-errors.ts** – Error handling
- **lib/stripe-webhook-handlers.ts** – Webhook handlers
- **components/stripe/stripe-provider.tsx** – Stripe Elements provider

## API Routes

| Route | Purpose |
|-------|---------|
| GET /api/stripe/config | Publishable key |
| POST /api/stripe/create-checkout-session | Checkout |
| POST /api/stripe/create-beta-checkout-session | Beta checkout |
| POST /api/stripe/create-portal-session | Customer portal |
| POST /api/stripe/create-customer | Create customer |
| POST /api/stripe/create-subscription | Create subscription |
| POST /api/stripe/create-free-subscription | Free subscription |
| POST /api/stripe/update-payment-method | Update payment method |
| POST /api/stripe/retry-payment | Retry payment |
| POST /api/stripe/webhook | Webhook handler |

## Environment Variables

- STRIPE_SECRET_KEY
- STRIPE_WEBHOOK_SECRET
- NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY
- STRIPE_PRICE_PLUS_YEARLY (optional, in stripe-config)
