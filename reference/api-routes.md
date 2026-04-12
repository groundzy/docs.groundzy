# API Routes Reference

All API routes are Next.js App Router route handlers in `app/api/`.

## Health

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/health` | Health check |

## Auth

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/auth/custom-token` | Create custom auth token (server-side) |

## User (Home / Plus)

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/user/sync-default-pro-share` | After setting a default certified pro with `defaultProOrganizationId`, fan-out tree access to that team and upsert a `clients` row on the pro org with contact + property summaries (Bearer token; body optional `{ organizationId }` must match user doc). Optional `{ homeownerPropertyIds: string[] }` limits mirroring and tree grants to those homeowner `properties` docs (non-empty array; must be ids the user owns). Omit for full scope. |
| POST | `/api/user/revoke-default-pro-share` | Before clearing default pro: removes org-based `tree_permissions` for that team (`grantSource` + `sourceOrganizationId`) on the homeowner’s trees; stamps CRM client stub if present (Bearer token; `{ organizationId }` must match current `defaultProOrganizationId`) |

## Invite & Teams

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/invite-code/validate` | Validate invite code |
| POST | `/api/teams/create-free-beta` | Create free beta team |

## AI

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/ai/chat` | AI chat (Gemini) |
| POST | `/api/ai/tree-summary` | AI tree summary |
| GET | `/api/ai/usage` | AI usage stats |

## Species

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/species/identify/reserve` | Reserve species identification slot |

## Trees

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/trees/archive` | Archive tree |
| POST | `/api/trees/transfer-ownership` | Transfer tree ownership |
| GET | `/api/trees/usage` | Tree usage stats |
| POST | `/api/trees/[treeId]/grant-user` | Authenticated tree owner: grant another user editor access (`tree_permissions` + `user_tree_permissions`) via Admin SDK; body `{ targetUserId }` |
| POST | `/api/trees/[treeId]/share-with-organization` | Authenticated tree owner: fan-out grant to all `teams/{organizationId}` members; body `{ organizationId? }` and/or `{ certifiedProId? }` (resolves `groundzy_pros.linkedOrganizationId`) |

## Quick Picks

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/quick-picks` | Get quick picks (requires composite indexes) |

## Weather

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/weather/current` | Current weather (WeatherAPI.com) |
| GET | `/api/weather/timeline` | Weather timeline (Visual Crossing) |

## Places & Pros

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/places/nearby-tree-services` | Nearby tree services (Google Places) |
| GET | `/api/groundzy-pros/nearby` | Nearby Groundzy pros |
| GET | `/api/resolve/pro-org` | Authenticated Home/Plus: resolve certified pro id → `linkedOrganizationId` |

## Quote portal (external homeowner)

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/quote-portal/issue` | Org member: issue portal token for a quote |
| GET | `/api/quote-portal/[token]` | Public: minimal quote summary for portal page |
| POST | `/api/quote-portal/[token]/claim` | Home/Plus: set `clients.homeownerUserId`, consume token |

## Image

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/image-proxy` | Proxy image from Firebase Storage |

## Share

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/share/[token]` | Resolve share token, return tree/share data |
| POST | `/api/share/[token]` | Redeem share link (Bearer token); grants access server-side |

## Stripe

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/stripe/config` | Stripe publishable key |
| POST | `/api/stripe/create-checkout-session` | Create checkout session |
| POST | `/api/stripe/create-beta-checkout-session` | Create beta checkout session |
| POST | `/api/stripe/create-portal-session` | Create customer portal session |
| POST | `/api/stripe/create-customer` | Create Stripe customer |
| POST | `/api/stripe/create-subscription` | Create subscription |
| POST | `/api/stripe/create-free-subscription` | Create free subscription |
| POST | `/api/stripe/update-payment-method` | Update payment method |
| POST | `/api/stripe/retry-payment` | Retry failed payment |
| POST | `/api/stripe/webhook` | Stripe webhook handler |

## Dev (development only)

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/dev/create-user` | Create dev user |
| POST | `/api/dev/simulate-payment` | Simulate payment |

## Environment Dependencies

| Route | Env vars |
|-------|----------|
| `/api/ai/chat` | `GOOGLE_GEMINI_API_KEY` or `GEMINI_API_KEY` |
| `/api/ai/tree-summary` | `GOOGLE_GEMINI_API_KEY` or `GEMINI_API_KEY` |
| `/api/weather/current` | `WEATHERAPI_KEY` |
| `/api/weather/timeline` | `VISUALCROSSING_API_KEY` |
| `/api/places/nearby-tree-services` | `GOOGLE_PLACES_API_KEY` or `GOOGLE_MAPS_API_KEY` |
| `/api/groundzy-pros/nearby` | `GOOGLE_PLACES_API_KEY` or `GOOGLE_MAPS_API_KEY` |
| `/api/image-proxy` | `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET` |
| `/api/stripe/*` | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `NEXT_PUBLIC_APP_URL` |
