# Decision log

**Purpose:** Record **significant** architectural and product decisions so future work stays aligned. Entries are **evidence-based** (file or doc references). Dates are included when known from git/docs; otherwise **Adopted (ongoing)**.

**Format:** Context → Decision → Consequences / notes.

---

## D-001 — Groundzy v3 non-negotiable principles

| Field | Value |
|-------|--------|
| **Status** | Adopted |
| **Source** | [`../00-foundation/principles.md`](../00-foundation/principles.md) |

**Context:** The legacy app accumulated multiple representations for history, workflow, and activity.

**Decision:** v3 commits to **one canonical model per concept**, **no duplicate systems** for history/workflow/activity as the long-term design, **explicit relationships** between CRM entities and inventory, and **tier clarity** (converge naming).

**Consequences:** New features must not reintroduce dual-write or parallel “mirror” patterns as permanent architecture; exceptions need explicit approval.

---

## D-002 — Stack: Next.js + Firebase + Stripe

| Field | Value |
|-------|--------|
| **Status** | Adopted (current app) |
| **Source** | [`../04-engineering/tech-stack.md`](../04-engineering/tech-stack.md), `firebase.json`, `apphosting.yaml`, `lib/stripe-config.ts` |

**Context:** Need a hosted web app with real-time data, auth, file storage, and subscriptions.

**Decision:** **Next.js** application; **Firestore** (rules in `firebase/firestore.rules`) for primary data; **Firebase App Hosting** for production; **Stripe** for billing and webhooks.

**Consequences:** Operational model tied to Firebase/ Google Cloud and Stripe dashboards; secrets via App Hosting secrets / env (see [`../10-operations/deployment.md`](../10-operations/deployment.md)).

---

## D-003 — Tree collaboration: permissions over `shared_data`

| Field | Value |
|-------|--------|
| **Status** | Adopted (migration direction) |
| **Source** | `lib/firebase/firestore.ts` (comments), [`../07-systems/permissions-system.md`](../07-systems/permissions-system.md), Firestore rules commentary |

**Context:** Older sharing used `shared_data`; collaboration moved toward explicit permission documents.

**Decision:** Prefer **`tree_permissions`** and **`user_tree_permissions`**; treat **`shared_data` as deprecated** for new writes.

**Consequences:** Docs and new code paths should not rely on `shared_data` as the primary model.

---

## D-004 — Job–tree association on Job documents

| Field | Value |
|-------|--------|
| **Status** | Adopted |
| **Source** | `types/tree.ts` (`activeJobIds` deprecated), `docs/features/current-workflow-audit.md` |

**Context:** Risk of inconsistent bidirectional sync if both Tree and Job store overlapping lists.

**Decision:** **Canonical** links live on **job** documents (`treeIds`, line items); **`Tree.business.activeJobIds`** is deprecated and not written by the app; tree-related workflow UIs **derive** related jobs by filtering scoped lists.

**Consequences:** Any code assuming `activeJobIds` on trees may be stale or empty.

---

## D-005 — CI quality gate without automated deploy

| Field | Value |
|-------|--------|
| **Status** | Adopted |
| **Source** | `.github/workflows/ci.yml` |

**Context:** Prevent broken builds on `main`.

**Decision:** On push/PR to **`main`**: Node 20, `npm ci`, **`lint`**, **`verify:drawers`** (warning), **`build`**. **No deploy job** in this workflow.

**Consequences:** Production deploy is **manual** or via other processes unless a separate pipeline is added.

---

## D-006 — Product glossary ownership

| Field | Value |
|-------|--------|
| **Status** | Adopted |
| **Source** | [`../01-product/core-concepts.md`](../01-product/core-concepts.md) |

**Context:** Avoid drift between docs and UI copy.

**Decision:** **Core concepts** file is the **single product glossary** for Tree, Zone, Property, Client, Work Item, Event, Workflow; other docs link instead of redefining.

**Consequences:** Terminology changes start in `core-concepts.md` and propagate.

---

## Pending / to formalize

| Topic | Note |
|-------|------|
| **Unified Event implementation** | Principles require one event story; legacy storage still has multiple paths until v3 data migration is complete ([`known-issues.md`](./known-issues.md)). |
| **Single tier naming model** | [`tier-system.md`](../01-product/tier-system.md) documents reconciliation; full convergence not yet recorded as a closed ADR. |

---

## Related

- [`glossary.md`](./glossary.md)
- [`known-issues.md`](./known-issues.md)
