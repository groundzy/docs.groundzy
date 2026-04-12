# Home / Plus ↔ Pro / Teams Linking Foundation

This document describes the **strategic base** for connecting **consumer-tier users** (Home, Plus) with **real Pro and Teams accounts** (Firebase users, organizations, billing). The product will expose many **entry points** over time (Hire a Pro, default pro, tree sharing, quotes/invoices, invites, and others). Those surfaces should plug into **shared primitives** so we do not duplicate identity logic or end up with incompatible “joins” between listings and accounts.

**Status:** Architecture and direction — implementation will evolve. For **what exists today** in code and Firestore, see the linked docs at the end.

---

## 1. Goals

1. **Resolve “pro as data” to “pro as account”**  
   Groundzy Certified Pros (`groundzy_pros`), Google Places businesses, and contact flows currently behave like **directories**: names, phones, place IDs, emails. Pro/Teams users live in **auth + org-scoped CRM data**. We need a durable way to know when a directory entry **is** (or **belongs to**) a given organization / Pro user. **Certified pro records** (including any future fields that link a listing to a Pro/Team org) are **created and edited in `admin.groundzy`** (the Groundzy admin application) — not in this consumer app (`app.groundzy`); operational and product work on that mapping belongs there alongside other admin-only data.

2. **Unify consumer–pro relationships**  
   “Default pro,” “shared tree collaborator,” “client created from quote,” and “contact request recipient” should all be able to reference the **same canonical Pro/Team identity** once known, instead of each feature storing unrelated snapshots (place ID only, email only, etc.).

3. **Support evolving acquisition channels**  
   Signup after quote/invoice, invite links, claim-your-listing, admin mapping for certified pros — these differ in **trust and signals** but should converge on **one resolution story**: weak identifier → verified link → `organizationId` (and optionally user ids).

4. **Preserve tier semantics**  
   Hire a Pro remains a **Home/Plus** surface; Pro/Teams users use CRM and workflow. The linking layer sits **between** tiers and must not blur security boundaries (see §7).

---

## 2. Terminology

| Term | Meaning |
|------|---------|
| **Consumer** | Authenticated user on Home or Plus (and similar non-business tiers). |
| **Pro / Teams account** | User with Pro or Teams subscription; Teams often has `organizationId` and team membership. |
| **Directory pro** | A professional **entity** shown in Hire Pro or Places: certified pro doc, or Places result — may not yet have a Groundzy login. |
| **Canonical pro identity** | The stable id Groundzy uses to say “this directory row and this org are the same business” (implementation TBD: e.g. org id + optional external keys). |
| **Resolution** | The process of attaching a **strong** account/org link to a **weak** identifier (place id, email, invite token). |
| **Relationship** | A consumer’s association with a pro: default pro, shared access, CRM client link, etc. |

---

## 3. Current state in the codebase (baseline)

This is the landscape the foundation must extend — not replace overnight.

### 3.1 Hire a Pro and directory data

- **Drawer:** `hire-groundzy-pro` — see [hire-pro.md](../features/hire-pro.md).
- **Groundzy Certified Pros:** Firestore `groundzy_pros` — admin-managed in **`admin.groundzy`** (Groundzy admin apps); this repo only reads them via `GET /api/groundzy-pros/nearby`.
- **Google Places:** `GET /api/places/nearby-tree-services` for additional businesses.
- **Unified UI type:** `NearbyPro` / certified variants in `types/certified-pro.ts` — includes `placeId`, optional `email`, `googlePlaceId`, etc.

### 3.2 Contact and default pro (consumer user document)

- **Contact:** `pro_contact_requests` — user-created; carries pro info for email/ops (see [firestore-collections.md](../reference/firestore-collections.md)).
- **Default pro:** `users/{uid}` fields `defaultProPlaceId`, `defaultProName` — optimized for **display and quick flows** (e.g. forms), keyed by **Place ID**, not by `organizationId`.

**Gap:** `defaultProPlaceId` does not require a Pro/Teams account to exist. That is correct for v1 but is exactly what the linking foundation should **upgrade** when an account exists.

### 3.3 Pro / Teams and organizations

- **Tiers:** `home` | `plus` | `pro` | `teams` (see signup and subscription logic in `app/actions/auth.ts`, tier utilities).
- **Teams:** `teams`, `team_members`, `invite_codes`; user doc `organizationId` — see [share-and-teams.md](../features/share-and-teams.md).
- **CRM and workflow:** Clients, properties, requests, quotes, jobs, invoices are **organization-scoped** (`organizationId`).

**Gap:** CRM “client” may represent a homeowner who **never** has a consumer account, or who **later** signs up. Linking should allow **eventually consistent** joins: same person/business, multiple signals.

### 3.4 Sharing (today)

- Tree sharing uses **tokens**, `shared_data`, and **userId** grants — see [share-and-teams.md](../features/share-and-teams.md).
- **Share with organization** (`POST /api/trees/[treeId]/share-with-organization`): after granting team members access, the server upserts a CRM client under the target org (`gz_hm_{organizationId}_{homeownerUid}`) with **`homeownerUserId`** set to the tree owner’s Firebase uid, mirrors the property when the tree has a `propertyId`, and writes **`tree.sharedCrmLinks[organizationId]`** so Pro UIs resolve client/property. Without **`clients.homeownerUserId`**, append-time workflow projections cannot add the homeowner’s `uid:` to `participantPrincipalIds`, so **Work** (participant list) stays empty for that consumer.
- Shared access is **not** yet framed as “share with this Pro org” as a first-class object; it is user-to-user. Future “share with my pro” flows should reuse **the same canonical org link** as Hire/default pro when possible.

---

## 4. Problem: weak identifiers vs strong accounts

| Layer | Examples | Stable for product logic? |
|-------|----------|---------------------------|
| **Weak** | Google `placeId`, formatted address, business name, phone | Good for search and display; **ambiguous** (branches, rebrands). |
| **Medium** | Email, `groundzy_pros` doc id, certified listing | Better; still may lack an account or duplicate businesses. |
| **Strong** | Firebase `uid`, `organizationId`, Stripe customer id | **Authoritative** for permissions, CRM, billing. |

The foundation’s job is to **record and upgrade** links from weak/medium → strong when evidence allows, without breaking existing **weak-only** flows.

---

## 5. Conceptual architecture (three layers)

Think of three cooperating layers. Exact collection names and fields are **product decisions**; the separation of concerns is what matters.

### 5.1 Canonical business / pro registry (directory ↔ account)

**Purpose:** Map external or curated identities to **zero or one** Groundzy organization (or dedicated “pro profile” id if you split hairs).

**Illustrative keys (not prescriptive):**

- `groundzy_pros/{proId}` ↔ optional `linkedOrganizationId`
- Google `placeId` ↔ optional `linkedOrganizationId` (possibly multi-step: claim flow)
- Email domain or verified email ↔ org (careful with shared inboxes)

**Operations:** Certified-pro curation and org linking in **`admin.groundzy`**; self-serve “claim my business”; invite-based linking after quote.

### 5.2 Consumer ↔ pro relationships

**Purpose:** Store **role-specific** edges that all point at the **same** canonical id when resolved.

Examples:

- **Default pro:** Beyond `defaultProPlaceId`, optionally `defaultProOrganizationId` (or derived from registry lookup).
- **CRM client:** `client` record with `homeownerUserId` or email match to consumer signup later.
- **Share / collaborate:** Grant or invite addressed to **org** or **pro user** instead of only username.

Each relationship type keeps its **own** rules (who can write, lifecycle). They share **references** to the canonical layer, not duplicate incompatible snapshots.

### 5.3 Resolution and verification channels

**Purpose:** How a weak link becomes strong — **policy**, not a single code path.

| Channel | Typical signal | Verification |
|---------|----------------|--------------|
| Certified pro admin (`admin.groundzy`) | Curated `groundzy_pros` row | Admin sets `linkedOrganizationId` (or equivalent) |
| Claim / verify listing | placeId + email / postcard / DNS | Automated or manual |
| Quote / invoice | Email on PDF, magic link | Token ties signup to org’s client record — see [quote-external-delivery-and-homeowner-signup.md](./quote-external-delivery-and-homeowner-signup.md) |
| Invite | `invite_codes` / team invite | Existing team flows |
| In-app (default pro) | User selects pro already linked | Immediate if registry has link |

Failed or partial resolution still leaves weak identifiers usable (current behavior).

---

## 6. Product surfaces (evolving)

These are **consumers** of the foundation — not separate identity systems.

1. **Hire a Pro / Contact**  
   Contact requests may later include **resolved org id** if the pro is linked, enabling in-app routing or CRM ingestion.

2. **Set default pro**  
   Prefer resolving to `organizationId` when available for downstream features (fast quotes, job handoff) while keeping `placeId` for Places-only businesses.

3. **Certified pros ↔ accounts**  
   Curated list should be the **easiest** path to a trusted mapping (admin + optional self-serve claim).

4. **Tree sharing**  
   Optional evolution: share with **organization** or **pro user**, not only username — requires canonical ids.

5. **Signup after quote / invoice**  
   Magic link or token associates new `uid` with existing **client** under an org; foundation ensures that client and consumer profile are **the same logical person** where intended. **Planned delivery channels** (SMS, email/PDF, print with QR) share one URL/token model — detailed in [quote-external-delivery-and-homeowner-signup.md](./quote-external-delivery-and-homeowner-signup.md).

6. **Future**  
   Referrals, partnerships, API integrations — same registry and relationship concepts.

---

## 7. Security, privacy, and permissions

- **Do not** expose org-internal CRM data to consumers solely because a placeId matched; resolution must respect **rules** and **consent**.
- **Pro-side** visibility: a consumer setting “default pro” does not automatically make them a **CRM client**; that may require accept flow or explicit job/quote relationship.
- **Auditability:** Changes to certified-pro ↔ org links should be traceable (who linked, when).
- **Firestore rules:** Any new fields or collections must preserve **tier boundaries** (consumer vs org-scoped reads/writes).

---

## 8. Implementation principles

1. **Backward compatible:** Existing `defaultProPlaceId`, `pro_contact_requests`, and Places-only pros keep working.
2. **Optional strengthening:** Add org ids where helpful; treat missing resolution as **normal**, not error.
3. **Single resolution API (conceptual):** Client features ask “resolve this placeId / certified id / email → org?” via a shared helper or server endpoint to avoid drift.
4. **Idempotent linking:** Repeated claims or signups should converge on one org link, not duplicate mappings.
5. **i18n and UX:** User-facing copy for “connect your business” / “link your account” should go through `lib/i18n` when shipped.

---

## 9. Tree operational surface (view-tree Ops) — v3 guardrails

Framing: the **tree** is the anchor; the Ops tab is one **view** over tree-level data, not a separate product silo. Long-term, “operational state” for a tree should come from a **canonical event (or equivalent) read model** shared across tiers—not from org-scoped workflow collections read through exceptions.

**Guardrails (non-negotiable for new work)**

- Do **not** build new **homeowner-facing workflow UI** on:
  - `work_items` (projections are not the product surface for homeowners),
  - `jobs` / `requests` / `quotes` / `invoices` via cross-org or homeowner-on-pro-org queries,
  - any **projection-only** layer as the user-visible source of truth.
- Future homeowner/pro operational visibility must originate from the **v3 canonical model** (e.g. events participation + tree relationship), not mirrors or rule gymnastics.

**Target model (v3):** Participant-based workflow access, global vs tree entry points, and identity rules are defined in [**Workflow participation & external access (v3)**](./workflow-participation-external-access-v3.md). That doc is the reference for *how* visibility should work once canonical records and participant permissions exist — it does not change the interim “no fake workflow UI on Ops” rule above.

**Shipped UX (consumer tiers)**

- Home / Plus **tree owners** without Teams pipeline: Ops shows **copy and CTAs** (Hire a Pro, Share tree)—not Teams upgrade as the primary path.
- **Pro** without Teams: copy + path to **Teams** for **their business** workflow (requests/quotes/jobs/invoices).
- Tree-scoped sections (property/client when present, service summary, recurring care, tags) are **not** gated behind Teams for display.
- There is **no** interim homeowner workflow status UI in Ops pending the canonical model (avoid “small card” regressions).

**QA matrix (manual, Phase 1)**

| Dimension | Cases |
|-----------|--------|
| Tier | Home, Plus, Pro without Teams |
| Role | Tree owner vs collaborator (shared access) |
| Property | Tree with vs without linked property |

Validate: correct primary card (consumer vs Pro solo), no pipeline work card without Teams, sections render as expected.

---

## 10. Open questions (to decide during rollout)

- **One org per business vs many locations:** Does one org represent a brand with many placeIds?
- **Pro tier without Teams:** Individual Pro may use `databaseCode` / non-org scoping — linking must specify whether targets are **user** vs **organization**.
- **Source of truth for Places:** If both certified and Places list the same business, merge rules?
- **Consumer account + CRM client:** Single user with both homeowner app and pro app (rare but possible) — identity merge policy.

---

## 11. Related documentation

| Document | Relevance |
|----------|-----------|
| [workflow-participation-external-access-v3.md](./workflow-participation-external-access-v3.md) | v3: participant access, global **Work** vs tree (events + links), canonical item views, identity — target model for homeowner/pro workflow visibility |
| [../features/hire-pro.md](../features/hire-pro.md) | Hire Pro UI, APIs, `groundzy_pros`, contact flow |
| [../features/share-and-teams.md](../features/share-and-teams.md) | Sharing, teams, `organizationId`, invites |
| [../reference/firestore-collections.md](../reference/firestore-collections.md) | `pro_contact_requests`, `groundzy_pros`, indexes |
| [../features/crm.md](../features/crm.md) | Clients, quotes, jobs — org-scoped workflows |
| [quote-external-delivery-and-homeowner-signup.md](./quote-external-delivery-and-homeowner-signup.md) | Planned: quote link/QR/token, external client view, homeowner signup + data linking |
| [../../Groundzy v3/06-features/hire-a-pro.md](../../Groundzy v3/06-features/hire-a-pro.md) | V3 feature summary |
| [../../Groundzy v3/05-data/data-flows.md](../../Groundzy v3/05-data/data-flows.md) | Hire Pro vs CRM request separation |

---

## Document history

| Date | Change |
|------|--------|
| 2026-04-04 | Initial architecture note; cross-link to quote external delivery / homeowner signup doc |
| 2026-04-04 | §9 Tree operational surface: v3 guardrails, shipped Ops UX, QA matrix |
| 2026-04-04 | §9 cross-link + related table: workflow-participation-external-access-v3.md (target participant model) |
