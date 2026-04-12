# Principles

These rules are **non-negotiable** for Groundzy v3 unless leadership explicitly revises them. They exist to prevent a repeat of legacy fragmentation (multiple history representations, mirrored workflow artifacts, drawer-by-drawer UX drift).

---

## Event-first system (canonical truth)

**Groundzy v3 is an event-first system.** **Events** are the **only** canonical source of truth for all actions in the system. Timelines, activity views, “work item”–shaped **projections**, dashboards, and commercial workflow UIs are **derived** from Events (and, where needed, from workflow documents for billing state)—not parallel authorities. No second storage layer may contradict the event stream as the record of what happened.

---

## 1. One system per concept

For each core domain idea—**tree**, **work**, **history / activity**, **commercial workflow**, **identity**, **billing**—there is **one canonical model** and **one primary persistence strategy**. **Facts (“what happened”) unify under Events** (see above). Projections, caches, and read models are allowed; **parallel sources of truth** are not.

- **Implication:** If two subsystems can disagree (e.g. “what happened” on a tree), the architecture is wrong until reconciled.

## 2. No duplicate systems for history, workflow, or activity

The legacy app combined **embedded tree history**, **event subcollections**, **`work_items`**, **CRM documents**, and **UI-only merges**. v3 must not reintroduce **dual-write** or **legacy mirror** patterns as the long-term design.

- **Implication:** A unified **event / timeline / work** story is designed **up front**, with explicit rules for tier, org, and privacy—not patched together with feature flags and ad hoc deduplication.

## 3. Consistent UI and UX

Users experience **one product**: shared shells, navigation patterns, density, and component vocabulary. Exceptions are **rare**, **documented**, and **reviewed**—not the default way new features ship.

- **Implication:** New surfaces compose from the **design system** and shared layouts; “special snowflake” drawers are a failure mode unless justified and contained.

## 4. System-first thinking over feature-first

We ship **coherent slices** of the system (data model + API + UI + permissions), not screens that imply behavior the backend does not guarantee.

- **Implication:** Roadmap items are framed as **capabilities** that strengthen the core graph (map ↔ trees ↔ clients ↔ work), not as isolated tickets that accumulate integration debt.

## 5. Explicit connectivity

Relationships between requests, quotes, jobs, invoices, trees, zones, and properties are **modeled and enforced**, not only passed through URLs or implied by navigation order.

- **Implication:** Conversions and state changes are **traceable** and **testable**—no reliance on “navigation-based prefills” as the only link between entities.

## 6. Tier clarity

Subscription tiers and org types are **aligned** across product copy, authorization, and data scoping. We do not maintain multiple conflicting naming schemes (e.g. signup vs runtime tier labels) without a migration path to **one** user-facing and code-facing model.

## 7. Maintainability at scale

Modules have **bounded size and responsibility**. Orchestration files that grow without bound are treated as **technical risk**, not inevitable complexity.

## 8. Legacy as input, not anchor

The existing repository is the **source of product knowledge and proven behavior**—not the template for v3 structure. We **extract** rules and flows; we **do not** copy fragmentation forward.

---

**Accountability:** Architecture reviews and PR standards reference these principles. Exceptions require **explicit** approval and a plan to converge back to the principle.
