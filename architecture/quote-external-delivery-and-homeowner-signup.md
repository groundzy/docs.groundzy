# Quote delivery to non–Groundzy clients: links, QR, and homeowner signup

**Status:** Planned — not implemented in `app.groundzy` yet. Complements [home-plus-to-pro-teams-linking.md](./home-plus-to-pro-teams-linking.md) (resolution channel: **quote / invoice → consumer account**).

---

## Problem

A Pro/Teams org creates **requests** and **quotes** for a **potential client** who does **not** have a Groundzy account. The business will eventually deliver quotes through multiple channels (examples):

| Channel | Example |
|---------|---------|
| Text / SMS | “View online” link |
| Email | HTML body + **PDF** attachment |
| Print | Paper handout |

We want **one coherent model**: optional **view quote** online, plus a **CTA to sign up** on Groundzy so the homeowner’s account can be **linked** to the existing CRM **client** (and quote context) without retyping data.

---

## Design principle: one canonical URL per delivery

Use a **single URL** for a given quote delivery (per issuance or per quote revision—product decision), regardless of channel:

- SMS → same URL  
- Email and PDF → same URL (and optional short code) in body and print layout  
- **QR code** on printed material → encodes that URL (or a short redirect that resolves to it)

Avoid three separate “integrations” (SMS vs email vs print). Differences are only **presentation** (mobile web, mail client, paper).

---

## Token model (server-side)

Expose **opaque, expiring tokens** backed by server-side storage (e.g. Firestore), **not** long URLs stuffed with PII or unsigned client-only payloads.

**Illustrative fields** (names are not prescriptive):

| Field | Purpose |
|-------|---------|
| `quoteId`, `organizationId`, `clientId` | Load quote in context for read-only view |
| `expiresAt` | Limit exposure window |
| `status` | `active` / `revoked` / `consumed` (if one-time claim) |
| Optional `channel` | `sms` \| `email` \| `print` — analytics only |

**Validation:** Server-side route (e.g. `GET /api/quote-portal/{token}` or dedicated page loader) validates token, returns minimal data for **that quote** and org branding — not broad CRM access.

---

## User flows

### 1. View quote (unauthenticated)

User opens link → sees **read-only** quote (and org identity/branding). No Groundzy login required for basic “customer received the quote” behavior.

### 2. Signup CTA (“Save in Groundzy” / “Track with [Arborist]”)

Same page includes CTA → signup/login with **token carried** through auth (query param, redirect state, or short-lived session cookie). After auth, a **trusted backend** associates `uid` with the CRM **client** record (e.g. `homeownerUserId` on `clients/{clientId}` or equivalent), aligning with the linking foundation’s **client ↔ consumer** resolution.

### 3. Data transfer after signup

- **Before signup:** Quote + client snapshot for display comes from org data via the token path.  
- **After linking:** Prefer **references** to CRM `clients` / `properties` where policy allows, or copy fields into consumer profile for homeowner UX (name, address, contact — product choice).  
- **Email mismatch:** If the user signs up with a different email than the client record, define explicit rules (verification step, manual confirm) — do not silently merge weak matches.

---

## Optional short code (print / SMS without tap)

Alongside QR and long URL, support a **human-enterable code** (e.g. `GZ-ABC123`) printed on PDFs and flyers. It resolves via the **same** token table (secondary key), not a second system.

---

## PDF-specific note

The PDF is a **carrier** for the URL + code. The PDF does not need a separate “signup integration” beyond what the link provides. Optional future: digital signatures, checksums — orthogonal to homeowner signup.

---

## Suggested rollout order

1. **Public (token-based) quote view** — high value, defines security boundary.  
2. **Post-auth linking** of `uid` ↔ `clientId` for the issuing org.  
3. **Richer features:** accept/decline in-app, deposits, shared trees, messaging — still reuse the same link + resolution story.

---

## Security

- Rate-limit token validation; prevent enumeration of quote ids.  
- Revoke or rotate tokens when quotes are superseded or withdrawn.  
- Scope: token grants access only to **that** quote (and minimal org branding), not the full CRM.  
- Audit: who issued tokens, optional access logs for compliance.

---

## Relation to CRM

Today, `quotes` and `clients` are **organization-scoped** (see [crm.md](../features/crm.md), `createQuote` / `createClient` in `lib/firebase/firestore.ts`). This document does not change that model; it adds an **external presentation + claim** layer on top.

---

## Related documentation

| Document | Relevance |
|----------|-----------|
| [home-plus-to-pro-teams-linking.md](./home-plus-to-pro-teams-linking.md) | Foundation: resolution channels, client ↔ consumer |
| [../features/crm.md](../features/crm.md) | Clients, quotes, workflow |
| [../features/share-and-teams.md](../features/share-and-teams.md) | Prior art: tokens and shared access patterns |

---

## Document history

| Date | Change |
|------|--------|
| 2026-04-04 | Initial plan: single URL/token, QR, SMS/email/print, post-signup linking |
