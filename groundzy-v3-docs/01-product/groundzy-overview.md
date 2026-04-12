# Groundzy Overview

This document frames **Groundzy** as a product and describes how the **current codebase** informs a planned **strategic rebuild (Groundzy v3)**. It is intended for internal planning, onboarding, and alignment.

**Product docs in this folder (single definitions, no duplication):**

| Doc | Purpose |
|-----|---------|
| [`core-concepts.md`](./core-concepts.md) | Canonical glossary: Tree, Zone, Property, Client, Work Item, **Event**, Workflow |
| [`tier-system.md`](./tier-system.md) | Plans, access, naming reconciliation |
| [`personas.md`](./personas.md) | Who uses Groundzy |
| [`user-journeys.md`](./user-journeys.md) | How users move through the product |

---

## What is Groundzy?

**Groundzy** is a map-centric web application for **smart tree care**. Users inventory and manage trees on an interactive map (Mapbox), maintain structured records (species, measurements, history, media), and use AI-assisted identification and chat. Weather-aware insights support planning. For tree care businesses, the product extends into **clients**, **properties**, and—on team tiers—a **requests → quotes → jobs → invoices** workflow.

The current application is organized as a **single-page, map-first experience**: a central map with **drawer-based** tools (dashboard, trees, CRM, AI, weather, settings, and more), with navigation driven in part by URL query parameters (`?drawer=…`) so views can be linked and revisited. See `docs/PROJECT_OVERVIEW.md` and `docs/architecture/complete-architecture-documentation.md` for how routing and layout are structured.

---

## Who is it for?

- **Homeowners and property stewards** — The **Home** tier is described in product copy as suited to mapping trees on one’s own property (`types/signup-flow.ts` tier descriptions).
- **Individual paying users (Plus, Pro)** — **Plus** adds higher usage limits and **Hire a Pro** (yearly Stripe path). **Pro** targets professionals, arborists, landscapers, and students who need expanded limits, export, clients/properties, and CRM-style surfaces.
- **Organizations and crews** — **Team** tiers (Small, Mid, Large, Enterprise) add collaboration, admin controls, dashboards, seasonal seats, and the full service workflow. Signup captures organization types such as arboriculture, landscaping, property management, municipalities, schools, and nonprofits.
- **Users seeking local help** — **Hire a Pro** (Home and Plus) connects users to nearby professionals and tree services; this drawer is **not** shown to Pro/Teams users, who are positioned as service providers (`docs/features/hire-pro.md`).

---

## The Problem Groundzy Solves

Tree care and property work are often **fragmented across tools**: photos in camera rolls, notes in documents, measurements in spreadsheets, and commercial follow-up in separate billing or CRM systems. Coordination between **what is on the ground** (which trees, where, what condition) and **what the business does** (quotes, jobs, invoices) is easy to lose.

Groundzy addresses this by:

- Anchoring inventory and context on a **shared map** with per-tree records and media.
- Layering **weather** and **AI** assistance for identification and guidance (tier-limited where applicable).
- For professionals and teams, attaching **clients and properties** and a **documented workflow pipeline** from inquiry through billing—so the same product can support both property-level stewardship and service operations.

---

## Why Groundzy Is Being Rebuilt

The current application was **built incrementally**: features and drawers were added over time, which is visible in the scale of the surface area (dozens of registered drawers, large orchestration files, and extensive domain logic spread across modules).

**Strategic motivation (product direction)**  
We now have a clearer picture of the end-state product. A rebuild is planned as a **reset in architecture and UX**, not a dismissal of what shipped: the goal is to deliver a **cohesive, seamless platform** that matches that vision, with consistent interaction patterns and reliable connectivity between subsystems.

**Pain themes (aligned with internal engineering documentation)**  
Internal docs describe real constraints that a greenfield approach can address holistically:

| Theme | What the repo reflects |
|--------|-------------------------|
| **Uneven cohesion** | The workflow audit documents stages and relationships but also lists **gaps, inconsistencies, and technical debt** (conversion semantics, status sync, duplicate helpers, type vs UI mismatches, map linkage limits). See `docs/features/current-workflow-audit.md` (noting: a separate **workflow upgrade** doc describes later improvements—treat the audit as historical analysis, not necessarily the live state of every line). |
| **UI/UX and maintainability at scale** | The drawer system doc scores **maintainability** below architecture and notes **megafiles**, registry comment drift, and ordering collisions. See `docs/audits/drawer-system-current-state.md`. |
| **Fragmented connectivity** | Workflow was initially **navigation + prefill**-heavy; upgrades moved toward relational conversions and shared line-item mappers (`docs/features/workflow-upgrade-implementation.md`). Remaining cross-cutting concerns (visibility, permissions, unified work items) appear in broader architecture plans (e.g. `docs/architecture/` master plans). |
| **Product model complexity** | Tier naming and signup models do not map 1:1 with runtime subscription types (e.g. `types/signup-flow.ts` vs `SubscriptionTier` in `lib/drawer-registry.ts`), which increases cognitive load for both users and implementers. |

Together, these factors support a **strategic rebuild**: carry forward validated domain knowledge while designing **unified flows**, a **consistent design system**, and **cleaner boundaries** between map, CRM, workflow, AI, and billing.

---

## Legacy Knowledge to Preserve

The existing repository should be treated as a **source of truth for product behavior**, not only as code to replace.

- **Business and access rules** — Subscription tiers, effective tier checks (`lib/utils/tier-utils.ts`), Stripe price mapping (`lib/stripe-config.ts`), and **Firestore / Storage rules** encode real constraints that must remain correct in any new system.
- **Validated workflows** — The **request → quote → job → invoice** pipeline, entity relationships, and status models are documented in workflow audits and implementation notes; these capture how the business actually operates in software.
- **Proven features** — Map + trees + zones + properties, AI surfaces, weather, search, sharing, teams, i18n, and billing integrations are all implemented and documented under `docs/features/` and `docs/PROJECT_OVERVIEW.md`.
- **Domain language** — Types under `types/`, adapters under `lib/workflow/`, and CRM entities reflect terminology crews already use.
- **Patterns worth evaluating for reuse** — Drawer **registry** + lazy loading, **Zustand** for UI/map state, **TanStack Query** for server data, **React Hook Form + Zod**, centralized **i18n**, and **Next.js API routes** + Firebase Admin patterns are proven; v3 may reimplement surfaces but can inherit **architectural lessons** and **integration boundaries** (what talks to Stripe, what lives in Firestore, etc.).

---

## Vision for Groundzy v3

**Groundzy v3** is the intended **end state** after a deliberate rebuild:

- **Unified platform experience** — One coherent mental model from map to CRM to workflow, with predictable navigation and fewer one-off interaction patterns.
- **Consistent design system** — Components, spacing, and states applied uniformly; fewer exceptions driven by historical drawer-by-drawer growth.
- **Seamless connectivity** — Entities and workflows stay linked across stages (requests, quotes, jobs, invoices, trees, zones, properties) with explicit, testable rules—not only URL handoffs.
- **Better architecture** — Clear module boundaries, smaller surface files where appropriate, and room for evolution (permissions, work-item unification, automation) without compounding incidental complexity.
- **Clearer product positioning** — Tiers and surfaces (e.g. homeowner vs pro vs team) explained consistently in product copy, data model, and UI.

This vision is **directional**; scope and phasing for v3 are product decisions outside this document.

---

## Key Features

**Domain terms** (Tree, Property, Client, Work Item, Event, Workflow) are defined only in [`core-concepts.md`](./core-concepts.md). Features are implemented across **many drawers** with tier-gated visibility (`lib/drawers.ts`, `lib/drawer-registry.ts`). Summary:

- **Map and geography** — Mapbox map, tree markers, zones, properties, drawing, measurement, filters.
- **Trees** — CRUD, species catalog, Quick Picks, media, history, AI identification.
- **AI** — Groundzy Wizard, Identifying Wand, AI Chat (Google Generative AI / Gemini); usage limits vary by tier.
- **Weather** — H3-based conditions, timeline, recommendations; presentation varies by tier.
- **CRM and workflow** — Clients, properties; requests, quotes, jobs, invoices on appropriate tiers.
- **Search** — Global search across trees, zones, properties, clients, species.
- **Sharing and teams** — Share links; team invites and roles.
- **Hire a Pro** — Home/Plus; nearby pros and Places-backed listings.
- **Plant Health Care (PHC)** — Documented planning area (`docs/features/plant-health-care.md`).
- **Internationalization** — English, Spanish, French.
- **Billing** — Stripe subscriptions synchronized with user documents.

---

## Tier System

See **[`tier-system.md`](./tier-system.md)** for the full tier matrix, **Home / Plus / Pro / Teams** behavior, naming inconsistencies between signup types and runtime `SubscriptionTier`, and paid-access rules. Summary: runtime tiers include **Home**, **Plus**, **Pro**, **Small Team**, **Mid Team**, **Large Team**, **Enterprise**; paid tiers depend on **active/trialing** Stripe status per `lib/utils/tier-utils.ts`.

---

## Technology Overview

| Layer | Technologies (current app) |
|-------|----------------------------|
| **Frontend** | Next.js 16+ (App Router), React 19, TypeScript |
| **UI** | shadcn/ui (Radix), Tailwind CSS 4, Lucide |
| **State** | Zustand (client/map/navigation); TanStack React Query (server data) |
| **Forms** | React Hook Form, Zod |
| **Maps / geo** | Mapbox GL, Mapbox Draw, Turf, h3-js |
| **Backend** | Firebase Auth, Firestore, Storage; Firebase Admin on server |
| **Payments** | Stripe (checkout, portal, webhooks) |
| **AI** | Google Generative AI (Gemini) |
| **Email** | Resend (where used) |
| **Hosting** | Firebase Hosting + Next deployment patterns in `docs/deployment/` |

v3 may **keep** the same ecosystem or adjust it; this table describes the **legacy stack** as implemented today.

---

## Notes / Unknowns

- **Audit vs current code** — `docs/features/current-workflow-audit.md` captures issues at a point in time; `workflow-upgrade-implementation.md` describes subsequent fixes. Before relying on any single audit bullet as “still broken,” verify against the latest code paths.
- **PHC scope** — Documented as a planning/feature area; full production scope should be confirmed against product priorities.
- **Enterprise / custom pricing** — Team metadata includes enterprise placeholders with null prices in `types/signup-flow.ts`; actual sales motion may be outside the app.
- **Future permission model** — Architecture docs reference evolving visibility/permission designs; v3 should align with whichever model is selected.
- **Rebuild delivery strategy** — This document does **not** specify whether v3 is a parallel codebase, strangler migration, or big-bang cutover—that is a program decision.

---

*Evidence for feature and tier descriptions: `docs/PROJECT_OVERVIEW.md`, `docs/features/README.md`, `types/signup-flow.ts`, `lib/drawer-registry.ts`, `lib/utils/tier-utils.ts`. Evidence for engineering tradeoffs: `docs/features/current-workflow-audit.md`, `docs/audits/drawer-system-current-state.md`, `docs/features/workflow-upgrade-implementation.md`.*
