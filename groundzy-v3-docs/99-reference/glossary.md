# Glossary

**Purpose:** One place for **definitions** used across Groundzy v3 docs. **Product-domain terms** (Tree, Property, Event, etc.) are **canonical** in [`../01-product/core-concepts.md`](../01-product/core-concepts.md)—this file **summarizes and extends** without redefining long prose there.

---

## Product domain (see core concepts)

| Term | Short definition |
|------|------------------|
| **Tree** | First-class mapped tree record: species, location, media, health context; may link to Property, Zone, Client. |
| **Zone** | Mapped polygon for spatial grouping above a single tree. |
| **Property** | Site / parcel record (address, boundaries, optional Client); org-scoped. |
| **Client** | CRM “who”: customer or account you serve. |
| **Work Item** | Product term for a unit of operational work; **v3:** projection/derived view from **Events** (and workflow docs where needed)—**not** a second source of truth. Legacy `work_items` may mirror CRM; v3 does not. |
| **Event** | **v3 canonical fact:** “something happened”; append-only; UI timelines are **views** of Events. |
| **Workflow** | Commercial pipeline **Request → Quote → Job → Invoice** (Teams-tier full pipeline in product matrix). |

**Diagram:** How concepts fit together — [`core-concepts.md`](../01-product/core-concepts.md#how-concepts-fit-together).

---

## Plans and access

**Full tier matrix, Plus/Pro/Teams behavior, and naming reconciliation:** [`../01-product/tier-system.md`](../01-product/tier-system.md) only. Below: quick pointers—**do not** duplicate capability tables here (anti-drift).

| Term | Meaning |
|------|---------|
| **Subscription tier** | Runtime: `SubscriptionTier` in `lib/drawer-registry.ts`. **Checks:** see tier-system § “Tier checks (v3).” |
| **Plan tier (signup)** | `PlanTier` in `types/signup-flow.ts` — **no Plus** in union; Plus exists at runtime only. |
| **Home / Plus / Pro / Teams** | Definitions and CRM/workflow entitlements — **tier-system.md**. |
| **Effective tier** | `getEffectiveSubscriptionTier` — paid tiers fall back to Home-equivalent gating if subscription inactive. |

**Reconciliation:** [`../01-product/tier-system.md`](../01-product/tier-system.md).

---

## Identity and collaboration

| Term | Meaning |
|------|---------|
| **User** | Authenticated account; profile and preferences under `users/{userId}` (and related paths per data model). |
| **Team** | Multi-user collaboration unit; `teams` collection; workflow defaults and org-scoped work. |
| **Organization** | Business scope for CRM, properties, workflow docs; `organizations` collection; user may carry `organizationId`. **Note:** “Team” and “organization” both appear in rules and types—see known issues. |
| **Tree sharing** | Access via `tree_permissions`, `user_tree_permissions`, share links, public summaries — **`shared_data` deprecated** in favor of tree permissions (Firestore rules / code comments). |

---

## Technical (app & data)

| Term | Meaning |
|------|---------|
| **Drawer** | Slide-over or bottom sheet UI for a feature or entity; registered in `lib/drawer-registry.ts`; tier-aware metadata. |
| **LEGACY_MIRROR** | Work item kind tied to mirrored CRM workflow rows (`types/work-item.ts`); not the long-term single source of truth per v3 principles. |
| **GZ TIN** | Groundzy tree identification number format (`GZ-…`) used when creating or converting trees (`lib/utils/gzTin.ts`). |
| **Job ↔ Tree link (canonical)** | Jobs reference trees via **`treeIds`** and line items; **`tree.business.activeJobIds`** on Tree is deprecated and not written by the app. |
| **Request (CRM)** | Workflow lead document (`requests` collection). Distinct from **`tree_access_requests`** (permission requests)—same word, different entity. |

**Collections overview:** [`../05-data/data-model-overview.md`](../05-data/data-model-overview.md).

---

## Integrations (short)

| Term | Role |
|------|------|
| **Firebase** | Auth, Firestore, Storage; Admin SDK on server; App Hosting for production deploy. |
| **Stripe** | Subscriptions and webhooks; tier entitlements tied to subscription status. |
| **Mapbox** | Map rendering and geocoding (client token). |

Details: [`../08-integrations/README.md`](../08-integrations/README.md).

---

## Related

- [`decision-log.md`](./decision-log.md)
- [`known-issues.md`](./known-issues.md)
