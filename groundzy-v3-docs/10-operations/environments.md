# Environments

**Canonical variable list:** `docs/reference/environment-variables.md`. **Local:** `.env.local` (not committed). **Production:** Firebase App Hosting secrets + `apphosting.yaml` env entries per `docs/deployment/DEPLOY.md`.

---

## Local development

| Aspect | Detail |
|--------|--------|
| **Command** | `npm run dev` — binds **0.0.0.0** for LAN/mobile testing (`package.json`) |
| **URL** | `http://localhost:3000` default |
| **Config** | Copy from `.env.example` pattern; fill `NEXT_PUBLIC_*` and server secrets as needed |

---

## Production (Firebase App Hosting)

| Aspect | Detail |
|--------|--------|
| **Secrets** | `npx firebase apphosting:secrets:set <NAME> --project <id>` — Stripe, Admin, Gemini, Places, weather, Resend, etc. (`DEPLOY.md` lists examples) |
| **Public env** | Firebase web config, Mapbox publishable token, Stripe publishable key, app URLs — often in `apphosting.yaml` **variable** entries |
| **URLs** | `NEXT_PUBLIC_APP_URL` (app links, Stripe return), `NEXT_PUBLIC_AUTH_APP_URL` (auth flows) |

---

## Stripe environments

- **Test vs live** keys and webhook secrets are **different** per Stripe mode.
- Webhook endpoint must match deployed domain: `https://<domain>/api/stripe/webhook`.

---

## CI environment

- **GitHub Actions** (`.github/workflows/ci.yml`): **Node 20**, `npm ci`, `lint`, `verify:drawers`, `build` on **push/PR to `main`**.
- **No** deployment step in the surveyed workflow—release is **manual** or separate pipeline unless added elsewhere.

---

## Scalability notes

- **`apphosting.yaml` `runConfig`** sets **maxInstances**, **concurrency**, **memory** — tune for traffic and cold starts (`minInstances: 0` = scale to zero when idle).
- **Firestore/Storage** scale with Firebase quotas; **indexes** required for heavy queries.

---

## Related

- [`deployment.md`](./deployment.md)
