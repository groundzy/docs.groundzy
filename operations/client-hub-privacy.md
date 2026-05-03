# Client hub — privacy and data handling (Phase 4B)

## What the hub session is

After `POST /api/client-hub/session/from-token` with a valid **document portal token**, the browser receives an HTTP-only cookie (`gz_client_hub`) signed with `CLIENT_HUB_SESSION_SECRET`. The cookie encodes:

- `organizationId` — the contractor’s org.
- `clientId` — the CRM client record linked to the quote/invoice/job/request behind that portal token.

No Groundzy staff or homeowner **Firebase user** is required for this MVP.

## Data shown

Hub APIs are **single-tenant**: they only return data for the cookie’s `organizationId` + `clientId` (the same contractor and CRM client the portal token proved access to). There is no cross-org merge.

- **GET /api/client-hub/documents** — merged list of **requests, quotes, invoices, and jobs** with titles, statuses, amounts, **property address snippets** where linked (including invoice→job→property resolution), and calendar hints (valid until, due date, scheduled work, request dates). Ordered primarily by `updatedAt` on the server.
- **GET /api/client-hub/trees** — **tree summaries** for trees tied to that client via CRM `clientId`, linked **properties**, or workflow line items (quotes/jobs/invoices/requests) for the same client. Payloads are client-facing only (species/nickname, coarse location, health headline, optional signed thumbnail URL). Internal staff notes are not exposed as a dedicated field.
- **GET /api/client-hub/trees/:treeId** — same guard as the list, one tree.
- **GET /api/client-hub/workflow-detail** — read-only summary (title, number, status, totals, line-item count, key dates) after verifying the entity belongs to the session; **PDF generation and quote approval** remain on the minted document portal (`POST /api/client-hub/mint-portal` + `/documents/{token}`).

## Sign-up CTA (marketing)

The hub UI may link to **Groundzy sign-up** (auth app or `/login` in local dev) with UTM parameters. **Using the hub does not require an account.** The sign-up destination sets cookies on the Groundzy app / auth domain per normal marketing flows.

## Retention and revocation

- Portal tokens remain governed by [`document-engine-phase3.md`](./document-engine-phase3.md) (TTL, revoke, approval consumption).
- Clearing the hub session: delete the `gz_client_hub` cookie (browser) or wait for expiry (default 24h).
- To **revoke all public access** for a client, staff should revoke active portal tokens (future admin UI) and rotate `CLIENT_HUB_SESSION_SECRET` if a cookie leak is suspected.

## PII minimization

- **Property addresses** appear as short labels when a workflow row or tree is linked to a property the org associates with that client—they are the same addresses the org already stores.
- Messaging (`client_hub_messages`) stores only text the client submits plus org/client ids and timestamps (see Firestore rules: client SDK has **no** read/write; only server routes).

## Rate limits

- Heavier hub reads (`trees`, `workflow-detail`) share a **hub_read** class in app rate limiting (see [`document-engine-phase4-hardening.md`](./document-engine-phase4-hardening.md) env pattern). Tune `PORTAL_RL_HUB_READ_MAX` / `PORTAL_RL_HUB_READ_WINDOW_MS` if needed.

## Production requirements

- Set a strong `CLIENT_HUB_SESSION_SECRET` (32+ characters) in production; the app returns **503** from session routes if it is missing in production.
