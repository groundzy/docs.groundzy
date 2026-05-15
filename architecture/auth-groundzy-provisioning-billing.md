# Auth.groundzy — provisioning and billing projection

This note complements the internal implementation plan (`auth` repo: `.cursor/plans/`).

## Runtime contract

- **`GET /api/account/status`** — Bearer Firebase ID token → `AccountReadyResponse` from `@groundzy/account-readiness`. Main app should call this after `/auth/callback` before deep-linking into routes.
- **`users/{uid}.provisioning`** — Orchestration hints (`status`, `schemaVersion`, `blockedCode`, timestamps). Dual-written after subscription changes via `applyProvisioningDualWriteAfterSubscriptionChange`.
- **`checkout_sessions/{sessionId}`** — Created when Stripe Checkout is created; includes `idempotencyKey`, `clientReferenceId` (Firebase `uid`), tier intent, return URL.
- **`stripe_webhook_events/{eventId}`** — Dedupe ledger; processing uses Stripe `events.retrieve` for idempotent replay.
- **`onboarding_drafts/{uid}`** — Debounced step autosave; **`users.provisioning.lastClientStep`** / **`completedSteps`** updated when the Firestore user profile exists.

## Operational scripts

- **`npm run reconcile-provisioning`** — Inspect/drain queued webhook mirrors and refresh provisioning dual-write cohorts (see script flags).
- **`npm run qa:stripe-webhook-replay`** — Replay a Stripe event id through the internal replay route (requires secrets in `.env.local`).

## Related runbooks (to add or extend)

- Webhook DLQ / replay
- Unknown Stripe price id (`repair_required`)
- Teams org repair (`POST /api/teams/repair` + onboarding repair panel)
