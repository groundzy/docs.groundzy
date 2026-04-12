# groundzy-v3-docs

Internal documentation for the Groundzy v3 rebuild. These docs define the architecture, data model, design system, and product decisions for the new system — and serve as the single source of truth to prevent the fragmentation that affected the legacy app.

---

## Purpose

- Align the team on critical decisions before and during the rebuild
- Document one coherent system (no parallel stories, no drift)
- Lock in irreversible architectural choices with explicit decision records

---

## Structure

| Folder | Contents |
|---|---|
| `00-foundation` | Mission, principles, vision, rebuild rationale, audit history |
| `01-product` | Core concepts, personas, tier system, user journeys |
| `02-design` | Design system, UI principles, component standards, interaction patterns |
| `03-architecture` | System overview, backend/frontend architecture, routing, state management |
| `04-engineering` | Tech stack, coding standards, naming conventions, file structure, testing |
| `05-data` | Data model, entities, relationships, event system, permissions, data flows |
| `06-features` | Per-feature docs: map, trees, workflow, CRM, search, AI, billing, teams |
| `07-systems` | Cross-cutting systems: event history, notifications, permissions, search |
| `09-business` | Pricing, tier rules, growth strategy |
| `10-operations` | Deployment, environments, monitoring |
| `99-reference` | Glossary, known issues, checklists, migration status |

---

## Locked Decisions

Three decisions are locked and may not be revisited without an explicit team decision:

- **D1 — Event-first:** Events are the only canonical source of truth. WorkItems are projections, not parallel records.
- **D2 — Access order:** Permission execution order is always: backend rules → org/roles → tier → UI.
- **D3 — Tier naming:** One canonical tier check pattern used across signup, Stripe, runtime UI, and analytics.

---

## Key Files

- [`00-foundation/v3-rebuild-alignment-plan.md`](00-foundation/v3-rebuild-alignment-plan.md) — Master roadmap for doc completion and system alignment
- [`05-data/tree-activity-v3-migration-status.md`](05-data/tree-activity-v3-migration-status.md) — Migration tracking from legacy embedded history to the event-append pipeline
- [`00-foundation/principles.md`](00-foundation/principles.md) — Core engineering and product principles

---

## Relationship to Legacy Docs

The `docs/` folder at the repo root contains legacy reference material and audit notes. Where v3 docs and legacy docs conflict, **v3 docs take precedence**. For example, `work-item-system-architecture.md` (legacy) treated WorkItems as a system of record — that framing is superseded by D1.
