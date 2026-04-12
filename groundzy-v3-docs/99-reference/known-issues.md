# Known issues

**Purpose:** Track **current problems and technical debt** visible from **repository evidence** (types, rules, docs). This is not a substitute for a live issue tracker—update as the codebase changes.

**Rule:** Prefer links to canonical v3 docs or source files over duplicating full analysis.

---

## 1. Data model and history

| Issue | Evidence | Impact |
|-------|----------|--------|
| **Multiple representations of tree activity** | Embedded `tree.history`, `tree_events` subcollections, operational `work_items`, and workflow mirrors; UI merge behavior influenced by preferences (`workItemsActivityTimelineEnabled`). | Complexity, risk of inconsistent timelines; v3 targets one Event concept per [`principles.md`](../00-foundation/principles.md). |
| **Workflow vs Work Item ambiguity** | CRM docs are billing-canonical; some `work_items` are **`LEGACY_MIRROR`** (`types/work-item.ts`); mirrors noted in [`data-model-overview.md`](../05-data/data-model-overview.md). | Unclear single source of truth for “what work happened” until unified model ships. |
| **Same word, different entities** | `Request` (CRM `requests`) vs **`tree_access_requests`** (sharing). | Confusion in docs and code search; glossary disambiguates ([`glossary.md`](./glossary.md)). |

**Detail:** [`../05-data/data-model-overview.md`](../05-data/data-model-overview.md) (§ Data problems), [`event-system.md`](../05-data/event-system.md).

---

## 2. Tier and product naming drift

| Issue | Evidence | Impact |
|-------|----------|--------|
| **Signup vs runtime tier types** | `PlanTier` in `types/signup-flow.ts` has **no Plus**; runtime uses **`SubscriptionTier`** with Plus and team size labels. | Types and copy can disagree; effective behavior from `tier-utils` and rules. |
| **Feature matrix vs pricing bullets** | [`tier-system.md`](../01-product/tier-system.md) documents README vs signup conflicts (e.g. tree limits). | Docs can contradict until one limits source exists. |
| **Team vs organization** | Both `teams` and `organizations` collections; user may use `organizationId`; rules mix org membership and tree permissions. | Mental model overhead for scoping and permissions debugging. |

---

## 3. Deprecated fields and APIs (still in types)

| Issue | Evidence | Impact |
|-------|----------|--------|
| **`Tree.business.activeJobIds`** | `@deprecated` in `types/tree.ts`; not written; jobs use `treeIds`. | Old data or assumptions may reference empty/stale lists. |
| **`shared_data` for sharing** | Firestore/rules direction + `firestore.ts` comments: prefer tree permissions. | Legacy paths may still exist in old documents. |
| **Request assignee** | `assignedToUserIds` preferred over deprecated single assignee (`types/request.ts`). | Migration of older requests if any. |
| **Tree health shortcuts** | Older health fields deprecated in favor of vigor/foliage/bark etc. (`types/tree.ts`). | Mixed records across tree age. |
| **Drawer registry helpers** | `getSidebarDrawerMetadata` / `getBottomNavDrawerMetadata` replace older APIs (`lib/drawer-registry.ts`). | Call sites should migrate. |

---

## 4. Access control stories

| Issue | Evidence | Impact |
|-------|----------|--------|
| **Tier vs org vs tree ACL** | [`data-model-overview.md`](../05-data/data-model-overview.md): subscription gates **UI**; many rules use **org** and **tree permissions**, not tier strings. | Two parallel stories—bugs can arise from assuming tier alone enforces data access. |

---

## 5. Operations and security

| Issue | Evidence | Impact |
|-------|----------|--------|
| **Secrets in repo** | `apphosting.yaml` may contain env values that should be **secrets-only** in ideal practice; [`deployment.md`](../10-operations/deployment.md) warns about committed credentials. | Credential leak risk; rotate if exposed. |
| **Limited in-repo observability** | Health route + optional OpenTelemetry hook; no full APM described in-repo ([`monitoring.md`](../10-operations/monitoring.md)). | Incidents rely on Firebase/Cloud/Stripe dashboards unless extended. |

---

## 6. Lint / tech debt (minor)

| Issue | Evidence | Impact |
|-------|----------|--------|
| **Intentional `eslint-disable`** | e.g. `react-hooks/exhaustive-deps`, `@next/next/no-img-element` in map/drawer components. | Documented exceptions; review if behavior bugs appear. |
| **`any` in drawer registry** | `lib/drawer-registry.ts` | Type safety gap in that module. |

---

## Resolved or out of scope here

- **Individual runtime bugs** — track in engineering workflow (issues/tests), not this static list unless they represent structural debt.

---

## Related

- [`glossary.md`](./glossary.md)
- [`decision-log.md`](./decision-log.md)
