# Groundzy v3 — documentation & architecture alignment plan

**Purpose:** Turn the v3 rebuild audit into **actionable decisions and doc work** so the documentation describes **one coherent system**, not parallel stories.

**Scope:** Groundzy v3 docs (`Groundzy v3/`), with pointers to legacy `docs/` and code where reconciliation is required.

**Out of scope here:** Implementation tickets, schema migrations, or code changes—this plan is alignment **first**.

**Status — D1:** LOCKED **Event-first** (2026-03-29). **Phase B:** checklist artifact [`99-reference/d1-pressure-test-checklist.md`](./99-reference/d1-pressure-test-checklist.md) (walkthrough not executed in docs). **Phase C (partial):** Plus/CRM fix in `tier-rules.md`; D2 execution order in `05-data/permissions.md`; D3 tier-check pattern in `tier-system.md`; CRM/workflow doc map in `workflow-system.md`; search cross-links; `permissions-system.md` slimmed to pointer + risks; glossary tier table shortened; drawer stance in `routing-and-navigation.md` + design-system link. **Remaining:** D2 policy matrix (detailed), link hygiene sweep, optional Phase B sign-off.

---

## Big picture: alignment plan vs system definition

This document is a **bridge**: it separates **system truth** (what Groundzy *is*) from **documentation cleanup** (how we *describe* it without drift).

- Finishing this plan produces a **buildable** spec—**if** the blocking decisions are written as **hard system rules**, not “we will decide later.”
- The goal is not only “a plan to decide things” but **an irreversible system definition** for v3, starting with **D1**.

---

## 1. Lock critical decisions (blocking)

Unresolved blocking decisions guarantee repeat fragmentation (dual-write, mirrors, merge-in-UI). **Almost everything downstream depends on D1.**

### D1 — Core system model (**DECIDED: Event-first**)

**Decision:** **Event-first.** **Events** are the **only** canonical source of truth for all actions. **WorkItems** (and legacy `work_items`) are **projections** or **derived** operational views—not a parallel system of record.

**Documented in:**

- `00-foundation/principles.md` — explicit event-first statement.
- `05-data/data-model-overview.md` — §4 “decided” table (no either/or).
- `05-data/event-system.md` — practical event shape + projection rules.
- `01-product/core-concepts.md` — Work Item as projection in v3 product language.
- `docs/architecture/work-item-system-architecture.md` — **superseded** for “WorkItem as system of record”; banner points to v3 docs.

**Rejected for v3:** WorkItem-first (Events as mere logs), or any blend where Events **and** WorkItems both act as system of record.

**Dependency note:** Data model, workflow narrative, history/activity, and much of UI structure **anchor on D1**. Permissions and tiers depend **partially** or **indirectly**.

---

### D2 — Access model (execution order + authority)

**Question:** How do **Firestore rules**, **org roles**, **subscription tier**, and **client UI** relate—without two conflicting “access stories”?

**Execution order** (locked in docs): see [`05-data/permissions.md`](./05-data/permissions.md) § **Execution order (authority)** — backend rules → org/roles → tier → UI.

**Remaining:**

- Full **policy matrix** (resource × action × who) aligned with `firebase/firestore.rules` — **not** yet written as a single table (optional follow-up in `permissions.md` or ops doc).
- `09-business/tier-rules.md` §3 already states tier is not security; cross-link to `permissions.md` if needed.

---

### D3 — Tier naming and a single check pattern

**Question:** One canonical set of labels for **signup**, **Stripe**, **runtime UI**, and **analytics**—or an explicit **mapping table** owned in-repo.

**Documented:** [`01-product/tier-system.md`](./01-product/tier-system.md) § **Tier checks (v3 — single pattern)**; [`99-reference/glossary.md`](./99-reference/glossary.md) plans section shortened to **pointer** to tier-system (no duplicate matrix).

**Remaining:** In-repo **mapping table** (PlanTier ↔ SubscriptionTier ↔ Stripe) when engineering reconciles types; optional ESLint / CI for “no ad hoc tier strings” — out of scope for markdown-only alignment.

---

## 2. Fix documented contradictions (urgent)

| Item | Status |
|------|--------|
| **Plus vs CRM** | Done — `09-business/tier-rules.md` §5 aligned with `clients-properties` tiers + `tier-system.md`. |
| **CRM vs workflow scope** | Done — **Documentation map** in `07-systems/workflow-system.md`. |
| **Principles vs core concepts** | Satisfied by **Event-first** D1 + `core-concepts.md` Work Item / Event wording; revisit only if product copy drifts. |

---

## 3. Reduce duplication and drift risk

**Anti-drift rule:** **No concept may have more than one authoritative definition.** One owner doc per concept; everywhere else links or adds only non-overlapping detail.

| Area | Owner doc | Other doc |
|------|-----------|-----------|
| **Permissions** | `05-data/permissions.md` | `07-systems/permissions-system.md` — pointer + risks only (done). |
| **Search** | Feature + system | `06-features/search.md` ↔ `07-systems/search-system.md` cross-links (done). |
| **Tiers / plans** | `01-product/tier-system.md` | `99-reference/glossary.md` — shortened (done). |

---

## 4. Legacy architecture doc disposition

| File | Plan |
|------|------|
| `docs/architecture/work-item-system-architecture.md` | **Banner added** (supersedes SoR claims). Optional later: **rewrite** body for historical accuracy only, or leave as legacy reference + ServiceType pointers. |

---

## 5. UI / navigation coherence (v3 target)

**Decision (locked for v3 docs):** **Drawers** are a **first-class navigation primitive** for the main authenticated product (map + work area), with URL-driven `drawer` params—not a disposable UI wrapper. See [`03-architecture/routing-and-navigation.md`](./03-architecture/routing-and-navigation.md) § **Drawers as system primitive (v3)**; [`02-design/design-system.md`](./02-design/design-system.md) §5 related table links there.

| Item | Action |
|------|--------|
| **Cross-links** | Optional sweep: `06-features/README.md`, `hire-a-pro` vs legacy paths — low priority. |

---

## 6. Verification checklist (before calling v3 “defined”)

- [x] D1 is **Event-first**: Events **only** as SoR; WorkItems as projections (`principles.md`, `data-model-overview.md` §4, `event-system.md` §3).
- [x] No “canonical **OR** absorbed” fork for Work Item in `data-model-overview.md`.
- [x] `tier-rules.md` and `tier-system.md` agree on **Plus** and **CRM/workflow**; spot-check `lib/drawers.ts`.
- [x] **Execution order** for access (D2) — see `permissions.md` § Execution order (full matrix still optional).
- [x] **Tier check pattern** (D3) — see `tier-system.md` § Tier checks.
- [x] `work-item-system-architecture.md` — **superseded** banner for SoR claims (full rewrite optional).
- [x] Glossary — tier table shortened; points to `tier-system.md`.
- [x] Drawer stance (§5) — `routing-and-navigation.md` + `design-system.md`.

---

## 7. Order of execution (do not skip)

**Phase A — D1 documentation** **(done for doc alignment)**

1. ~~Make the D1 call~~ **Event-first** (locked).
2. ~~Update: `principles.md`, `data-model-overview.md`, `event-system.md`, `core-concepts.md`~~; legacy `work-item-system-architecture.md` banner added.

**Phase B — Pressure-test D1**

- Use [`99-reference/d1-pressure-test-checklist.md`](./99-reference/d1-pressure-test-checklist.md).
- Walk scenarios: workflow conversions, tree ↔ job ↔ line items, activity timelines, team vs solo.
- Confirm relationships are expressible **without** reintroducing mirrors or dual-write as the long-term design.
- **Sign-off** row in checklist when complete.

**Phase C — Rest of this plan** (substantially advanced in docs)

3. ~~Contradiction fixes~~ · 4. ~~D2 order~~ (matrix optional) · 5. ~~D3 pattern + glossary~~ · 6. ~~Dedupe permissions/search~~ · 7. ~~Drawer stance~~ · link hygiene optional.

Do **not** treat documentation cleanup as a substitute for completing **Phase B walkthrough** (human/session).

---

## Related

- `Groundzy v3/00-foundation/principles.md`
- `Groundzy v3/05-data/data-model-overview.md`
- `Groundzy v3/99-reference/known-issues.md`
- `Groundzy v3/99-reference/d1-pressure-test-checklist.md`
- Internal audit conversation (rebuild alignment review)
