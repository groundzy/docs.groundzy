# Deployment Guide

This document covers setup, environment variables, and deployment for the Groundzy app.

## Prerequisites

- Node.js 18+
- npm
- Firebase CLI (`npm install -g firebase-tools`)
- Firebase project (create at [Firebase Console](https://console.firebase.google.com))
- Stripe account (for payments)

## Environment Variables

Create `.env.local` in the project root. See [reference/environment-variables.md](../reference/environment-variables.md) for the full list.

### Required (minimum to run)

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_FIREBASE_API_KEY` | Firebase web API key |
| `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN` | Firebase auth domain |
| `NEXT_PUBLIC_FIREBASE_PROJECT_ID` | Firebase project ID |
| `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET` | Firebase storage bucket |
| `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` | Firebase messaging sender ID |
| `NEXT_PUBLIC_FIREBASE_APP_ID` | Firebase app ID |
| `NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN` | Mapbox access token (maps, geocoding) |

### Server-only (secrets)

| Variable | Description |
|----------|-------------|
| `FIREBASE_ADMIN_PRIVATE_KEY` | Firebase Admin private key (from service account JSON) |
| `FIREBASE_ADMIN_CLIENT_EMAIL` | Firebase Admin client email |
| `STRIPE_SECRET_KEY` | Stripe secret key |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook signing secret |

### Optional (features)

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Stripe publishable key (client) |
| `GOOGLE_GEMINI_API_KEY` / `GEMINI_API_KEY` | Google Gemini (AI chat) |
| `NEXT_PUBLIC_PLANTNET_API_KEY` | PlantNet (species ID) |
| `GOOGLE_PLACES_API_KEY` / `GOOGLE_MAPS_API_KEY` | Google Places (nearby tree services) |
| `VISUALCROSSING_API_KEY` | Visual Crossing (weather timeline) |
| `WEATHERAPI_KEY` | WeatherAPI.com (current weather) |
| `RESEND_API_KEY` | Resend (transactional emails) |
| `RESEND_FROM_EMAIL` | From email for Resend |
| `NEXT_PUBLIC_APP_URL` | App URL (Stripe redirects, emails) |
| `NEXT_PUBLIC_AUTH_APP_URL` | Auth app URL (sign-in redirects) |

## Local Development

```bash
npm install
npm run dev
```

App runs at [http://localhost:3000](http://localhost:3000). For network access (e.g. mobile testing), the dev server binds to `0.0.0.0` by default.

## Firebase Setup

1. **Create project** in Firebase Console
2. **Enable** Authentication (Email/Password, Google, etc.), Firestore, Storage
3. **Create web app** and copy config values to `.env.local`
4. **Create service account** (Project Settings → Service accounts → Generate new private key) and set `FIREBASE_ADMIN_*` env vars
5. **Deploy rules and indexes**:
   ```bash
   npx firebase use <project-id>
   npx firebase deploy --only firestore
   npx firebase deploy --only storage
   ```

## Stripe Setup

1. Create products and prices in Stripe Dashboard
2. Set price IDs in `lib/stripe-config.ts` (or env vars where used)
3. Configure webhook endpoint: `https://<your-domain>/api/stripe/webhook`
4. Add `STRIPE_SECRET_KEY` and `STRIPE_WEBHOOK_SECRET` to env

## Firebase App Hosting (Production)

The project uses Firebase App Hosting. Configuration is in `apphosting.yaml`.

1. **Set secrets** (run from project root):
   ```bash
   npx firebase apphosting:secrets:set STRIPE_SECRET_KEY --project <project-id>
   npx firebase apphosting:secrets:set STRIPE_WEBHOOK_SECRET --project <project-id>
   npx firebase apphosting:secrets:set FIREBASE_ADMIN_CLIENT_EMAIL --project <project-id>
   npx firebase apphosting:secrets:set FIREBASE_ADMIN_PRIVATE_KEY --project <project-id>
   # Optional:
   npx firebase apphosting:secrets:set GEMINI_API_KEY --project <project-id>
   npx firebase apphosting:secrets:set GOOGLE_PLACES_API_KEY --project <project-id>
   npx firebase apphosting:secrets:set VISUALCROSSING_API_KEY --project <project-id>
   npx firebase apphosting:secrets:set WEATHERAPI_KEY --project <project-id>
   npx firebase apphosting:secrets:set RESEND_API_KEY --project <project-id>
   npx firebase apphosting:secrets:set PLANTNET_API_KEY --project <project-id>
   ```

2. **Deploy**:
   ```bash
   npm run build
   firebase deploy
   ```

Or use the full deploy script:
```bash
npm run deploy
```

## npm Scripts

| Script | Description |
|--------|-------------|
| `npm run dev` | Start dev server (binds to 0.0.0.0) |
| `npm run build` | Production build |
| `npm run start` | Start production server |
| `npm run lint` | Run ESLint |
| `npm run lint:fix` | Run ESLint with auto-fix |
| `npm run deploy` | Build + firebase deploy |
| `npm run deploy:hosting` | Build + deploy hosting only |
| `npm run deploy:firestore` | Deploy Firestore rules/indexes only |
| `npm run deploy:storage` | Deploy Storage rules only |
| `npm run verify:api` | Verify API routes (node scripts/verify-api-routes.mjs) |
| `npm run generate:favicon` | Generate favicon (node scripts/generate-favicon.mjs) |
