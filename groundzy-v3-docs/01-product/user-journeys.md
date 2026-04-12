# User Journeys

How people **move through Groundzy** at a product level. Terminology matches [`core-concepts.md`](./core-concepts.md). Tiers and gates are detailed in [`tier-system.md`](./tier-system.md).

---

## 1. First-time setup

1. Authenticate (Firebase Auth in the current app).
2. Land on the main **map + work area** shell.
3. Complete profile/preferences; subscription tier determines which drawers and limits apply.
4. Optional: join a **team** via invite code (team tiers).

**Outcome:** User has a home dashboard/map context scoped to their account (and **organization** when applicable).

---

## 2. Map-first tree stewardship (all relevant tiers)

1. Open **map**; pan/zoom to a site.
2. **Add a tree** (or multiple): place marker, species, measurements, photos.
3. **View tree**: overview, **Activity** (events/history presentation), media, system info—tab pattern in current app.
4. Optionally draw a **zone** or link to **property** context when the account has those features.

**Outcome:** **Trees** (and optionally **zones**) exist as map-anchored records; **Events** accumulate over time as the unified activity story (v3); legacy app may merge multiple storage paths in UI.

---

## 3. AI and weather assist

1. Open **AI** surfaces (Wizard, Identifying Wand, Chat—per tier limits).
2. Use **weather** context for planning (presentation varies by tier).

**Outcome:** Decisions informed by tools; usage may be capped on lower tiers.

---

## 4. Hire a professional (Home / Plus)

1. Open **Hire a Pro** (not shown to Pro/Teams users as service providers—`docs/features/hire-pro.md`).
2. Set location, browse nearby pros or listings, send contact request.

**Outcome:** Human help outside the user’s self-service inventory—does not require CRM **Workflow**.

---

## 5. Professional CRM without full pipeline (Pro)

1. Create and manage **Clients** and **Properties**.
2. Associate **Trees** with properties/clients as the product allows.
3. Operate lists, search, and property/tree detail without the full **Request → Quote → Job → Invoice** drawer set (per `docs/features/README.md`).

**Outcome:** Commercial context on the map without the full team workflow surface in the tier matrix.

---

## 6. Full commercial workflow (Teams)

1. Capture lead as **Request** (with client/property/tree context as applicable).
2. Produce **Quote** → convert or proceed to **Job** → **Invoice**.
3. Tie work to **Trees**, **Properties**, and **Clients**; track status on each entity.

**Outcome:** End-to-end service lifecycle in-product, aligned with **Workflow** in [`core-concepts.md`](./core-concepts.md).

---

## 7. Collaboration and sharing

1. **Team**: invite members, assign roles, use org-scoped data.
2. **Share**: generate share links for trees or related content where implemented; recipients may see restricted views.

**Outcome:** Multi-user operations and controlled visibility.

---

## 8. Account and billing

1. Upgrade/downgrade tier; Stripe manages paid plans in the current implementation.
2. **Effective access** for paid tiers depends on subscription status (`lib/utils/tier-utils.ts` behavior—active/trialing).

**Outcome:** Feature set matches paid state; see [`tier-system.md`](./tier-system.md) for naming and inconsistencies.

---

## Related

- [`personas.md`](./personas.md) — who takes these journeys
- [`groundzy-overview.md`](./groundzy-overview.md) — product summary and feature list
