# Client hub — privacy and data handling (Phase 4B)

## What the hub session is

After `POST /api/client-hub/session/from-token` with a valid **document portal token**, the browser receives an HTTP-only cookie (`gz_client_hub`) signed with `CLIENT_HUB_SESSION_SECRET`. The cookie encodes:

- `organizationId` — the contractor’s org.
- `clientId` — the CRM client record linked to the quote/invoice/job behind that portal token.

No Groundzy staff or homeowner **Firebase user** is required for this MVP.

## Data shown

- **GET /api/client-hub/documents** returns a merged list of quotes, invoices, and jobs for that `organizationId` + `clientId` (same data the org already stores in Firestore), limited and ordered by `updatedAt`.

## Retention and revocation

- Portal tokens remain governed by [`document-engine-phase3.md`](./document-engine-phase3.md) (TTL, revoke, approval consumption).
- Clearing the hub session: delete the `gz_client_hub` cookie (browser) or wait for expiry (default 24h).
- To **revoke all public access** for a client, staff should revoke active portal tokens (future admin UI) and rotate `CLIENT_HUB_SESSION_SECRET` if a cookie leak is suspected.

## PII minimization

- Hub APIs do not return full client addresses or internal notes beyond what is already on quote/invoice/job list projections used for titles/status/amounts.
- Messaging (`client_hub_messages`) stores only text the client submits plus org/client ids and timestamps (see Firestore rules: client SDK has **no** read/write; only server routes).

## Production requirements

- Set a strong `CLIENT_HUB_SESSION_SECRET` (32+ characters) in production; the app returns **503** from session routes if it is missing in production.
