# D1 pressure-test checklist (Event-first)

**Purpose:** Validate that **Event-first** (single canonical source of truth for actions) holds under real product flows—**before** scaling implementation. Use this in design review or engineering walkthroughs.

**Related:** [`../00-foundation/principles.md`](../00-foundation/principles.md), [`../05-data/event-system.md`](../05-data/event-system.md), [`v3-rebuild-alignment-plan.md`](../v3-rebuild-alignment-plan.md) Phase B.

---

## Pass criteria (each scenario)

For every row below, confirm **long-term v3 target** (not legacy quirks):

1. **One write path** to canonical **Events** for the user-visible action (no required dual-write to a second authority).
2. **No mirror rows** as a parallel source of truth (legacy `workflow_*` work_items pattern is **not** recreated).
3. **No UI-only stitching** as the only link between domains (relationships are **data**, then reflected in UI).

If a scenario **cannot** pass without violating (1)–(3), capture the gap as an **explicit exception** with a plan to converge—not an undocumented shortcut.

---

## Inventory & trees

| # | Scenario | Questions |
|---|----------|-----------|
| B1 | **Create tree** | What **Event type(s)** are emitted? What is **never** written elsewhere as a second “truth”? |
| B2 | **Edit tree** (species, location, health) | Same as B1; edits vs full replaces; idempotency if relevant. |
| B3 | **Tree media** | Upload/delete: Event + Storage reference; single story for “what changed.” |
| B4 | **Activity / timeline** | Timeline = **query/projection** over Events—confirm no parallel embedded history required for correctness. |

---

## Zones & properties

| # | Scenario | Questions |
|---|----------|-----------|
| B5 | **Zone create/edit** | Events for geometry or metadata changes; link to org/property where applicable. |
| B6 | **Property ↔ client ↔ tree** | Relationship changes expressed as Events and/or FK updates **without** a second narrative store. |

---

## CRM & workflow (Teams)

| # | Scenario | Questions |
|---|----------|-----------|
| B7 | **Request → Quote → Job → Invoice** | Each **transition** recorded as **Event(s)**; commercial docs hold billing state; **no** new mirror collection for “the same” transition. |
| B8 | **Convert request to quote / quote to job / job to invoice** | Backlinks (`convertedTo*`) + Events; no orphan conversions. |
| B9 | **Attach trees (and zones) to job / line items** | `treeIds` / line items + Events for attachment changes; tree and job stay linkable in queries. |
| B10 | **Job status change** (schedule, complete, cancel) | Event per meaningful transition; aligns with state machine docs. |

---

## Solo vs team / org

| # | Scenario | Questions |
|---|----------|-----------|
| B11 | **Solo user** (`databaseCode` / personal org) | Events scoped correctly; no accidental team-only paths. |
| B12 | **Team member** | `organizationId`, roles: Events include org scope; rules remain authoritative (see [`../05-data/permissions.md`](../05-data/permissions.md)). |

---

## Dashboard & “upcoming work”

| # | Scenario | Questions |
|---|----------|-----------|
| B13 | **Upcoming / scheduled** | Derived from Events + workflow state (or single projection table **fed from** Events)—not a second authoritative schedule store that can disagree with Events. |

---

## Sign-off

| Role | Name | Date | Notes |
|------|------|------|-------|
| Product / architecture | | | |
| Engineering lead | | | |

**Outcome:** Pass / Pass with documented exceptions / **Blocked** (list blockers).
