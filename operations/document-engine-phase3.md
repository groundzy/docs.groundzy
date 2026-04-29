# Document engine — Phase 3 (client portal links)

## Overview

Staff can mint **document portal tokens** stored in Firestore at `document_portal_tokens/{token}`. Clients open **`/documents/{token}`** (no Groundzy login) to view summary metadata, download a freshly generated PDF (`?includePdf=1` on the public API), and **approve quotes** when the token includes the `approve_quote` scope.

Tokens are created only through **server routes** (Admin SDK). Firestore rules deny all client reads/writes on `document_portal_tokens` (same pattern as `quote_portal_tokens`).

## Token lifetimes

- Default TTL: **90 days** from issuance (see `DEFAULT_TTL_MS` in `lib/documents/portal-token.ts`).
- **Revoked** tokens (`status: "revoked"`) return HTTP 410 from the public API.
- **Quote approval**: after a successful `POST /api/quotes/approve-by-token`, the token doc receives `approvalConsumedAt`. The link remains valid for viewing; a second approval attempt returns **409**.

## Staff APIs

| Route | Auth | Purpose |
|--------|------|---------|
| `POST /api/documents/portal-issue` | Firebase ID token | Mint token; returns `portalUrl` (`/documents/{token}`). |
| `POST /api/documents/sms-link` | Firebase ID token | Mint token + send SMS via Twilio (returns **503** if SMS env not set). |
| `GET /api/documents/public/{token}` | None | Read-only portal payload; add `?includePdf=1` for a short-lived signed PDF URL. |

## SMS (optional)

Set these environment variables on the app host:

- `TWILIO_ACCOUNT_SID`
- `TWILIO_AUTH_TOKEN`
- `TWILIO_FROM_NUMBER` (E.164, e.g. `+15551234567`)

If any are missing, `POST /api/documents/sms-link` responds with **503** and message `SMS not configured`.

## Resend (email)

Phase 3 does not change Resend usage; quote/invoice PDFs are still emailed via existing `POST /api/documents/email` when staff uses the send dialog.

## Troubleshooting

- **404 on public link**: token doc missing, or entity deleted / org mismatch on the token payload builder.
- **410 inactive**: token revoked or past `expiresAt`.
- **PDF fails**: check server logs for `renderAndStoreDocument` / Storage signing; ensure the service account can sign URLs or fall back to `storage.googleapis.com` URLs per `lib/documents/storage.ts`.
- **Approve 409**: quote already approved or `approvalConsumedAt` already set on the token.

## Manual QA matrix (smoke)

1. Issue portal for quote → open `/documents/{token}` → totals match drawer.
2. Download PDF (`includePdf=1`) → PDF opens in new tab.
3. Approve with name + optional signature → quote shows approved in app; second approve returns 409.
4. Invoice portal → no approve button; PDF downloads.
5. Job work order from **View job** and **Jobs list** → PDF opens.
6. Expired token (adjust `expiresAt` in emulator/admin) → 410.
7. SMS route with env unset → 503; with test credentials → 200 and message received (staging only).
