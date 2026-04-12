# Monitoring & reliability

**Reality check:** The repo exposes a **minimal** operational surface for observability; full APM may live outside the codebase (Firebase Console, Google Cloud, Stripe Dashboard).

---

## What exists in-repo

| Mechanism | Location | Role |
|-----------|----------|------|
| **Health endpoint** | `GET /api/health` | Returns `{ ok: true, timestamp }` — **liveness** for load balancers or uptime checks |
| **OpenTelemetry API** | `@opentelemetry/api` in `package.json`; `next.config.ts` serverExternalPackages | **Optional** instrumentation hook—no full tracing config guaranteed in repo |
| **Client errors** | `console.error` patterns in libs | Ad hoc; not a centralized error reporting SDK in surveyed files |

---

## Platform-native monitoring (operational)

| Platform | Use |
|----------|-----|
| **Firebase Console** | Auth errors, Firestore usage, Storage, App Hosting logs |
| **Google Cloud Logging** | Often tied to App Hosting / Cloud Run backend for Next |
| **Stripe Dashboard** | Webhook delivery, payment failures, subscription health |
| **Mapbox / external APIs** | Vendor dashboards for quota and errors |

---

## CI as quality gate

- **`ci.yml`:** lint + build on each PR to `main` — catches broken builds before deploy.
- **`verify:drawers`:** Warning-only; does not block unless workflow is tightened.

---

## Reliability practices (from architecture)

- **Stripe webhooks:** Must be **idempotent**-safe under retries (Stripe behavior).
- **Firestore rules:** Primary **data security** monitor—misconfiguration shows as denied writes in client logs.
- **Indexes:** Missing index → query failures; deploy `firestore.indexes.json` when Firestore suggests URLs in errors.

---

## Gaps (v3 ops backlog)

| Gap | Suggestion |
|-----|------------|
| **Centralized logging** | Structured logs + correlation ids for API routes |
| **Alerting** | Uptime on `/api/health`, Stripe webhook failure rates, error budget |
| **RUM / performance** | Web Vitals or Firebase Performance if not already enabled in project |

---

## Related

- [`deployment.md`](./deployment.md)
- [`environments.md`](./environments.md)
