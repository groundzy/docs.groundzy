# Deployment

**Sources:** `docs/deployment/DEPLOY.md`, `firebase.json`, `apphosting.yaml`, `package.json` scripts.

---

## Hosting model

- **Firebase** hosts the Next.js app via **Firebase App Hosting** with **`firebase.json`** `hosting` + **`frameworksBackend`** (region `us-central1` in current config).
- **Deploy command:** `npm run deploy` → `npm run build && firebase deploy` (`package.json`).
- **Partial deploys:** `deploy:hosting`, `deploy:firestore`, `deploy:storage` for targeted updates.

---

## App Hosting configuration

- **`apphosting.yaml`** defines **runConfig** (e.g. min/max instances, concurrency, CPU, memory) and **env** variables for the hosted backend.
- **Secrets** (Stripe, Firebase Admin, third-party APIs) should be set with **`firebase apphosting:secrets:set`** per `DEPLOY.md`; reference secret names in yaml rather than embedding production values in git.

**Operational risk:** Ensure **no live secrets** are committed to the repo. Rotate any credentials that appear in tracked files.

---

## Pre-deploy checklist (typical)

1. `npm run build` succeeds locally or in CI.
2. Firestore **indexes** deployed if queries changed (`firebase deploy --only firestore`).
3. **Stripe webhook** URL points to production `/api/stripe/webhook`.
4. **Environment** / secrets aligned with target project (see [`environments.md`](./environments.md)).

---

## Related scripts

| Script | Purpose |
|--------|---------|
| `verify:api` | Validates API route wiring (`scripts/verify-api-routes.mjs`) |
| `verify:drawers` | Non-blocking drawer line-count / discipline (`scripts/drawer-pr-check-warn.mjs`) |

---

## Related

- [`environments.md`](./environments.md)
- [`monitoring.md`](./monitoring.md)
- `Groundzy v3/08-integrations/*` (external services to configure per env)
