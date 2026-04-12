# Rebuild Rationale

## Why we are building Groundzy v3

The current Groundzy application **validated the market and the domain**. It shipped real map-centric tree care, AI assistance, weather, CRM, team workflow, billing, and sharing. That work is **valuable and preserved as product knowledge** in the legacy codebase and documentation.

The rebuild is **strategic**, not reactive. We are not starting over because the legacy app “failed.” We are rebuilding because **incremental delivery over time**—necessary to learn quickly—produced **structural consequences** that a mature product can no longer afford to carry indefinitely.

---

## What incremental build cost us

Evidence from the repository and internal docs includes:

- **Fragmentation of concepts** — Multiple representations of “what happened” and “what work exists” (e.g. tree history, events, work index rows, CRM documents, UI merges), requiring adapters and dual-writes instead of a single coherent story.
- **Uneven cohesion** — Workflow and CRM behavior documented with real gaps (conversion semantics, status alignment, duplicate helpers) in historical audits; subsequent patches improved specific paths but did not erase the underlying **integration-by-accretion** pattern.
- **UI/UX and maintainability drift** — Dozens of surfaces, documented shell exceptions, megafiles, and registry inconsistencies: architecture can score well while **consistency and maintainability** lag.
- **Product model complexity** — Tier and plan naming that does not map 1:1 across signup data, runtime subscription types, and user-facing copy increases load on both users and engineers.

These are **systemic** outcomes of **feature-first growth**, not individual bad decisions.

---

## Why a full rebuild (v3)

A **greenfield Groundzy v3** lets us:

- Apply **one system per concept** and **one primary history/work activity model** (see `principles.md`).
- Implement **consistent UI/UX** from a shared design system and a small set of archetypes—not retroactively impose shell rules on years of varied drawers.
- **Reconnect** map, inventory, CRM, and workflow with **explicit** relationships and testable rules.
- **Simplify** tier and org semantics for users and for authorization code.

The legacy app remains the **reference implementation** for behavior, edge cases, and business rules until v3 replaces it. It is **not** the template for v3’s internal architecture.

---

## Summary

| Statement | Commitment |
|-----------|------------|
| Legacy validated the problem and much of the solution | **Preserve** domain knowledge and workflows as requirements. |
| Incremental build caused fragmentation | **Design v3** to unify concepts and connectivity by default. |
| Rebuild is strategic | **Invest** in foundation—data model, UX system, and principles—before scaling new surface area. |

For a detailed audit of history and UI fragmentation in the current repo, see [`rebuild-audit-history-and-uiux.md`](./rebuild-audit-history-and-uiux.md) in this folder.
