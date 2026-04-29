# Document engine — Phase 4A hardening (rate limits, PDF cache, logs)

## Rate limits (`lib/server/portal-rate-limit.ts`)

Sliding-window counters are **in-memory per Node instance**. On Vercel/serverless, traffic spreads across isolates, so limits are **best-effort** and should be paired with:

- Vercel Firewall / WAF rules, or
- Redis/Upstash-backed limits (not bundled in this repo).

Environment overrides (optional):

| Variable | Default | Meaning |
|----------|---------|---------|
| `PORTAL_RL_PUBLIC_META_MAX` | 120 | Max public metadata GETs per window per `IP:tokenPrefix` |
| `PORTAL_RL_PUBLIC_META_WINDOW_MS` | 60000 | Window for metadata |
| `PORTAL_RL_PUBLIC_PDF_MAX` | 20 | Max `includePdf=1` requests per window |
| `PORTAL_RL_PUBLIC_PDF_WINDOW_MS` | 60000 | Window for PDF |
| `PORTAL_RL_APPROVE_IP_MAX` | 30 | Approve-by-token per IP per window |
| `PORTAL_RL_APPROVE_TOKEN_MAX` | 10 | Approve-by-token per portal token per window |
| `PORTAL_RL_PORTAL_ISSUE_MAX` | 60 | Staff portal-issue per Firebase uid per window |
| `PORTAL_RL_SMS_MAX` | 20 | SMS-link per uid per **3h window** (see `PORTAL_RL_SMS_WINDOW_MS`, default 3_600_000) |
| `PORTAL_RL_HUB_MINT_MAX` | 15 | Client-hub mint-portal per session key per hour |
| `PORTAL_RL_HUB_SESSION_MAX` | 20 | Client-hub session-from-token per IP per minute |

Stable **429** JSON: `{ "error": "Too many requests", "code": "rate_limited" }`.

## PDF cache (`document_portal_tokens`)

After a successful public PDF generation, the token doc may store:

- `lastPdfStoragePath` — GCS object path under the org bucket.
- `lastPdfEntityUpdatedAtMs` — millis from the source quote/invoice/job `updatedAt` (fallback `createdAt`).

On `GET /api/documents/public/[token]?includePdf=1`, if the entity revision matches, the API **re-signs** the existing object instead of calling `renderAndStoreDocument` (cache hit). Any quote change updates `updatedAt`, invalidating the cache.

**Approve-by-token** clears `lastPdfStoragePath` and `lastPdfEntityUpdatedAtMs` on the token so the next PDF reflects approval.

## Observability

`[document-portal]` JSON lines via `logPortalEvent` in `lib/documents/portal-log.ts` (portal-issue, public GET, PDF hit/generate, approve-by-token, SMS, client-hub events). Ship logs to your log platform and alert on 5xx spikes and PDF duration.

## Client hub session (Phase 4B)

- `CLIENT_HUB_SESSION_SECRET` — required in **production** (min 32 chars) for signed `gz_client_hub` cookies issued by `POST /api/client-hub/session/from-token`.

## Rollback

- Disable aggressive limits by raising env max values.
- PDF cache: delete `lastPdf*` fields on affected token docs (Admin) to force regeneration.
