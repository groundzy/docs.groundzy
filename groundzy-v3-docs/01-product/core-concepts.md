# Core Concepts

**This file is the single product glossary for Groundzy.** Other product docs use these terms without redefining them. If a term changes here, update downstream docs in the same change.

---

## Tree

A **Tree** is a first-class record for an individual tree (or mapped tree entity) tied to a location on the map: species, measurements, media, health-related fields, and links to stewardship context.

- **Belongs to** a scoped data context (individual user database and/or **organization**, depending on account type).
- **May link to** a **Property**, **Zone**, and/or **Client** when the account uses CRM concepts.
- **Activity** over time (care actions, notes, inspections, measurements) is represented in the product as **Events** (see below)—not as multiple parallel “history products.”

*Legacy implementation note:* The current app stores tree activity in embedded history arrays and/or `tree_events` paths; v3 targets **one Event model** (see `Groundzy v3/00-foundation/principles.md`).

---

## Zone

A **Zone** is a mapped area (polygon) used to group or describe land at a level above a single tree—often for planning, bulk operations, grouped-tree care, or spatial context.

- Zones relate to **Trees** and may anchor **work** scoped to an area.
- A zone can represent a **grouped-tree care record**: aggregate inventory counts (`speciesCounts`), individually assigned trees (`tree.zoneId`), or both.
- Trees physically inside a polygon are spatial candidates; they are not considered intentionally managed by the zone until assigned or represented in aggregate inventory.
- Product docs treat “zone” consistently as spatial grouping, not as a substitute for **Property** or **Client**.

---

## Property

A **Property** is a **site** record: address, optional boundaries, optional link to a **Client**, and metadata (name, type, notes). It is the commercial/stewardship anchor for “this parcel or site” as distinct from a single **Tree**.

- In the data model, properties are **organization-scoped** and may optionally reference a `clientId` (`types/property.ts`).
- **Trees** can be associated with a property so inventory and service context align.

---

## Client

A **Client** is a **customer or account** you do business with: individual, business, government, or nonprofit contact and communication details.

- **Properties** belong to the service story (“where”) while **Clients** belong to the relationship story (“who”).
- Team and Pro surfaces use clients for CRM lists, requests, quotes, jobs, and invoices.

---

## Work Item

A **Work Item** is **one unit of operational work** in product language: something to plan, schedule, execute, or track—whether it originates from tree/zone care, a recurring plan, or the commercial pipeline.

- **Groundzy v3:** Work Items are **not** a second source of truth. They are **projections** or **derived views** (scheduling lists, dashboards, “upcoming”) built from **Events** and workflow documents where needed—see [`Groundzy v3/00-foundation/principles.md`](../00-foundation/principles.md).
- **Legacy:** `work_items` included **mirrored** rows tied to CRM documents; that pattern is **not** carried forward as canonical.
- **Product language:** users think in terms of **work** (scheduled prune, inspection, job block), not “database rows”—**Work Item** remains the label for that **concept** in UX; storage is event-first.

---

## Event (unified activity / history model)

An **Event** is the **canonical, append-only fact** that something happened at a point in time: e.g. service performed, note added, inspection recorded, measurement taken, workflow stage advanced, or access granted—subject to **tier**, **role**, and **privacy** rules.

**Rules for Groundzy v3 product language:**

- There is **one Event concept** for “what happened” across trees, properties, org activity, and workflow—implemented as **views and permissions** on top of a unified model, not as separate unrelated “histories.”
- **Timeline** and **Activity** in the UI are **presentations** of Events (filters and grouping), not separate sources of truth.
- **Public** or **community** summaries may show **redacted or coarse Events** for safety; they are still projections of the same conceptual Event, not a second parallel history product.

*Legacy note:* The current app merges multiple storage paths in the UI for Activity; v3 replaces that with a **single conceptual Event** per `principles.md`.

---

## Workflow

**Workflow** is the **commercial service pipeline**: **Request → Quote → Job → Invoice** (and related statuses), connecting **Clients**, **Properties**, **Trees**/line items, and billing.

- **Workflow** is **not** the same as “any work”—day-to-day tree **Events** and **Work Items** can exist outside a full commercial workflow on appropriate tiers.
- Team tiers expose the full workflow drawers and entities; Pro focuses on **clients and properties** without that full pipeline per `docs/features/README.md` tier matrix.

---

## How concepts fit together

```
Map
 ├── Zone (optional area)
 ├── Property (site / parcel context)
 │     └── Client (who you serve)
 └── Tree (individual tree record)
        └── Events (canonical — unified activity / history)
        └── Work Items (derived projections for scheduling & ops UX; not parallel truth)

Workflow (commercial): Request → Quote → Job → Invoice
```

---

## Related

- Tiers and gating: [`tier-system.md`](./tier-system.md)
- Who uses the product: [`personas.md`](./personas.md)
- Flows: [`user-journeys.md`](./user-journeys.md)
