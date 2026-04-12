# Personas

Personas are inferred from product copy and structure in the repository (`types/signup-flow.ts`, `docs/features/README.md`, `docs/features/hire-pro.md`, CRM and tier docs). They are **roles**, not separate products—see [`core-concepts.md`](./core-concepts.md) for shared definitions.

---

## Property steward (Home-focused)

**Who:** Homeowner or land steward mapping and caring for trees they are responsible for.

**Goals:** See trees on a map, record basics, use AI help and weather, stay within free-tier limits; optionally **hire a professional** for work they do not self-perform.

**Typical tier:** **Home**; may upgrade to **Plus** for higher limits and Hire a Pro.

**Key interactions:** Dashboard, map, add/view trees, AI identify/chat (limits), weather, profile, **Hire a Pro** (where offered). Does not use full CRM or commercial **Workflow** in the tier matrix.

---

## Practicing professional (Pro)

**Who:** Arborist, landscaper, student, or solo operator who needs a professional inventory and client/site context without the full team **Workflow** pipeline in the feature matrix.

**Goals:** Unlimited or expanded tree work, export, **Clients** and **Properties**, CRM-style lists; prioritize jobs in the field without necessarily running full request→invoice in-app.

**Typical tier:** **Pro** (paid individual).

**Key interactions:** Everything in steward-focused flows where applicable, plus clients/properties, expanded AI usage, operational views. **Hire a Pro** is **not** aimed at this persona in the product (they are the service provider).

---

## Team operator (Teams)

**Who:** Crews, companies, municipalities, schools, nonprofits—organizations that need collaboration, admin, and the full commercial spine.

**Goals:** Same domain objects as Pro, plus team coordination, dashboards, and **Workflow**: **requests → quotes → jobs → invoices**.

**Typical tier:** **Small Team**, **Mid Team**, **Large Team**, or **Enterprise** (pricing and seats vary; see [`tier-system.md`](./tier-system.md)).

**Key interactions:** CRM **Workflow** drawers, team settings, invites, role-based access, work tied to **Clients**, **Properties**, and **Trees**.

---

## Groundzy-guided helper seeker

**Who:** User who wants to find a tree care professional near them (not building a CRM).

**Goals:** Discover and contact pros or services; may overlap with Property steward on **Home** or **Plus**.

**Typical tier:** **Home** or **Plus** (Hire a Pro surfaces).

**Note:** This is a **journey** layered on the steward persona, not a separate data model.

---

## Internal / admin (implicit)

**Who:** Org owner, admin, or manager managing seats, invites, and settings.

**Goals:** Control membership, visibility where the product supports it, and seasonal or team limits per plan.

**Typical tier:** **Teams** (and org-scoped features).

---

## Related

- Canonical entities: [`core-concepts.md`](./core-concepts.md)
- End-to-end flows: [`user-journeys.md`](./user-journeys.md)
