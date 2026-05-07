# Integrations hub (add-ons)

## Purpose

The **Integrations** drawer (`integrations`) is the product surface for third-party connections and future add-ons. It is **tier-scoped at two levels**:

1. **Hub access** — Only **Pro** and **team** subscription tiers can open the drawer (same audience as the workflow pipeline). Home and Plus users do not see it in navigation; a direct URL shows an upgrade prompt.
2. **Per-integration access** — Each catalog entry may require a `CapabilityKey` and/or a minimum tier group (`pro` | `teams`). Locked entries show upgrade CTAs (for example **Upgrade to Teams**).

## Code map (app repo)

| Area | Location |
|------|----------|
| Drawer entry | `app/drawers/integrations.tsx` |
| Registry | `lib/drawers.ts` (`visibleForTiers: WORKFLOW_TIERS`, desktop sidebar hidden like Settings) |
| Catalog | `lib/integrations/catalog.ts` — add new rows here; wire sections via `hubSection` |
| Hub tier helper | `lib/utils/tier-utils.ts` — `canAccessIntegrationsHub()` |
| Capabilities | `lib/capabilities.ts` — per-integration keys (e.g. `integrations.bouncie_fleet` Teams-only; `integrations.stripe_connect` Pro + teams, aligned with `connect-payments-eligibility`) |
| Discovery | Unified Settings → Organization (workflow users), desktop map hub account menu |
| i18n | `lib/i18n/messages.ts` — `integrations.*`, `navigation.drawerLabels.integrations`, `mapHub.menuIntegrations` |

## Adding an integration

1. Add a `CapabilityKey` if the feature is tier-gated; grant it only in `getCapabilitiesForTier` for the tiers that should **use** the integration (not necessarily every hub viewer).
2. Append a row to `INTEGRATION_CATALOG` with `hubSection`, `status`, and optional `requiredCapability` / `minimumTierGroup` / `availableAction` (navigate to an existing drawer, e.g. Settings → Organization → Client payments for Stripe).
3. Add `integrations.catalog.<id>.*` (and `integrations.actions.*` when using `availableAction`) in all locales in `messages.ts`.
4. Implement connect flows, APIs, and map/UI behavior; keep the hub honest (no fake “Connect” until OAuth/storage exists). **Stripe** and **Bouncie** ship real connect flows; placeholders should use `setupComingSoon`.

## Bouncie fleet (Teams)

**Product:** Settings → Organization → **Bouncie fleet tracking**; map toolbar **Fleet** filter (`mapToolbar.filters.fleet` / capability `integrations.bouncie_fleet`).

**Developer portal:** Register the app at [Bouncie Developer Portal](https://www.bouncie.dev/), note [Bouncie API](https://docs.bouncie.dev/) (OAuth + webhooks + REST `https://api.bouncie.dev/v1/`).

**Environment (app repo):** See root `.env.example` — `BOUNCIE_CLIENT_ID`, `BOUNCIE_CLIENT_SECRET`, `BOUNCIE_WEBHOOK_SECRET`, `BOUNCIE_OAUTH_STATE_SECRET` (random HMAC secret for signed OAuth `state`). `NEXT_PUBLIC_APP_URL` must match the OAuth redirect base (`/api/bouncie/oauth/callback`).

**Firestore (server-written):**

- `teams/{teamId}.bouncieFleet` — public metadata (`connected`, `bouncieUserId`, …); clients cannot patch (`firestore.rules`).
- `teams/{teamId}/bouncie_private/auth` — refresh/access tokens; **no client access**.
- `teams/{teamId}/bouncie_vehicle_locations/{id}` — lat/lng markers; **read** for team members, **write** via Admin only.
- `teams/{teamId}/bouncie_webhook_dedupe/{hash}` — idempotency for webhooks; **no client access**.
- `bouncie_account_routing/{accountId}` — routes webhooks to `organizationId`; **no client access**.

**API routes (Next.js handlers under app repo):** `app/api/bouncie/oauth/start`, `oauth/callback`, `disconnect`, `status`, `sync`, `webhook`.

## Related

- [Drawer PR checklist](../audits/DRAWER_PR_CHECKLIST.md)
- [Drawer shell classification](../audits/drawer-shell-classification.md)
- [Capabilities / tiers](../reference/pricing-reference.md) (pricing reference if needed)
