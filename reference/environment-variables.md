# Environment Variables Reference

Create `.env.local` in the project root for local development. For production (Firebase App Hosting), use `apphosting.yaml` and secrets.

## Firebase (Client)

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_FIREBASE_API_KEY` | Yes | Firebase web API key |
| `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN` | Yes | Auth domain (e.g. project.firebaseapp.com) |
| `NEXT_PUBLIC_FIREBASE_PROJECT_ID` | Yes | Firebase project ID |
| `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET` | Yes | Storage bucket |
| `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` | Yes | Messaging sender ID |
| `NEXT_PUBLIC_FIREBASE_APP_ID` | Yes | Firebase app ID |

## Firebase Admin (Server)

| Variable | Required | Description |
|----------|----------|-------------|
| `FIREBASE_ADMIN_PRIVATE_KEY` | Yes* | Service account private key (from JSON; replace `\n` with newlines) |
| `FIREBASE_ADMIN_CLIENT_EMAIL` | Yes* | Service account client email |

*Required for: custom tokens, Stripe webhooks, share resolution, quick-picks seed, dev create-user.

## Stripe

| Variable | Required | Description |
|----------|----------|-------------|
| `STRIPE_SECRET_KEY` | Yes* | Stripe secret key |
| `STRIPE_WEBHOOK_SECRET` | Yes* | Webhook signing secret |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Yes* | Publishable key (client) |
| `STRIPE_PRICE_PLUS_YEARLY` | No | Price ID for Plus yearly (fallback in stripe-config) |

*Required for payment flows.

## Mapbox

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN` | Yes | Mapbox access token (maps, geocoding, address autocomplete) |

## URLs

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_APP_URL` | No | App URL (default: https://app.groundzy.com). Used for Stripe redirects, emails. |
| `NEXT_PUBLIC_AUTH_APP_URL` | No | Auth app URL (default: https://auth.groundzy.com). Used for sign-in redirects. |

## AI

| Variable | Required | Description |
|----------|----------|-------------|
| `GOOGLE_GEMINI_API_KEY` | No | Google Gemini API key (AI chat, tree summary) |
| `GEMINI_API_KEY` | No | Alias for GOOGLE_GEMINI_API_KEY |

## Species Identification

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_PLANTNET_API_KEY` | No | PlantNet API key (species ID from client; add domain to PlantNet Authorized domains) |

## Weather

| Variable | Required | Description |
|----------|----------|-------------|
| `VISUALCROSSING_API_KEY` | No | Visual Crossing API key (weather timeline) |
| `WEATHERAPI_KEY` | No | WeatherAPI.com key (current weather) |

## Places

| Variable | Required | Description |
|----------|----------|-------------|
| `GOOGLE_PLACES_API_KEY` | No | Google Places API key (nearby tree services, Groundzy pros) |
| `GOOGLE_MAPS_API_KEY` | No | Alias |
| `GOOGLE_API_KEY` | No | Alias |

## Email

| Variable | Required | Description |
|----------|----------|-------------|
| `RESEND_API_KEY` | No | Resend API key (transactional emails) |
| `RESEND_FROM_EMAIL` | No | From email (default: Groundzy &lt;noreply@app.groundzy.com&gt;) |

## Intelligence & internal jobs (server)

| Variable | Required | Description |
|----------|----------|-------------|
| `GROUNDZY_INTELLIGENCE_ACTOR_UID` | Yes* | Firebase Auth **user** uid used as `actorUserId` on `groundzy_events` for system-generated `intelligence.*` appends (dedicated service account or bot user). |
| `GROUNDZY_INTERNAL_CRON_SECRET` | Yes* | Shared secret for `POST /api/internal/intelligence/*` (send as `x-groundzy-internal-secret` or `Authorization: Bearer ...`). |
| `STORM_INTEL_BATCH_SIZE` | No | Default **150** — max trees per `run-batch` Firestore query (clamped to 1–500 in the API). |
| `STORM_INTEL_BATCH_MAX_MS` | No | Default **180000** (3 minutes) — soft time budget per batch invocation; stop early so the route stays under hosting limits. The `run-batch` route sets `maxDuration` to **300** seconds (Next.js / Firebase App Hosting). Override per request with JSON body `maxDurationMs` (capped at 290000). |

*Required when calling intelligence append or internal evaluation routes in that environment.

See also: [`storm-intelligence-batch-cron.md`](../groundzy-v3-docs/07-systems/storm-intelligence-batch-cron.md) (batch endpoint, cursor, Cloud Scheduler).

## Development

| Variable | Required | Description |
|----------|----------|-------------|
| `NODE_ENV` | Auto | `development` \| `production` |
| `BASE_URL` | No | Base URL for verify-api-routes script (default: http://localhost:3000) |
| `VERCEL_URL` | Auto | Set by Vercel; used as fallback for `NEXT_PUBLIC_APP_URL` |

## Example .env.local (minimal)

```env
# Firebase
NEXT_PUBLIC_FIREBASE_API_KEY=...
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your-project
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your-project.firebasestorage.app
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=...
NEXT_PUBLIC_FIREBASE_APP_ID=...

# Mapbox
NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN=pk....

# Optional: Stripe (for payments)
STRIPE_SECRET_KEY=sk_...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_...

# Optional: Firebase Admin (for server-side features)
FIREBASE_ADMIN_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
FIREBASE_ADMIN_CLIENT_EMAIL=firebase-adminsdk-xxx@your-project.iam.gserviceaccount.com
```
