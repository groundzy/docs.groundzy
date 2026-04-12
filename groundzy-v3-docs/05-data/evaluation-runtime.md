# Evaluation runtime (targeted re-evaluation)

**Status:** Specification — where and how intelligence rules run; **not** “evaluate everything on every tick.”  
**Depends on:** [`intelligence-rules.md`](./intelligence-rules.md), [`../07-systems/alert-notification-system.md`](../07-systems/alert-notification-system.md).

---

## 1. Principle

**Targeted re-evaluation:** each trigger loads **minimal context** and runs **only the rule subset** registered for that trigger class.

Avoid:

- Full table scans of all trees on every cron
- Running Pro-only rules for Home tier (see [`../09-business/intelligence-tier-matrix.md`](../09-business/intelligence-tier-matrix.md))

---

## 2. Trigger taxonomy

### 2.1 External / ingestion

| Trigger | Typical fan-out | Rule subset |
|---------|-----------------|-------------|
| Forecast updated for H3 cell | Trees whose location maps to cell (or property centroids in cell) | Weather + combined |
| Significant weather change | Same | Combined (higher priority) |

### 2.2 Internal (domain events)

Examples (names align with `GroundzyEventType` when applicable):

| Trigger | Fan-out | Rule subset |
|---------|---------|-------------|
| `tree.inspection_recorded` | That tree + org | Tree + combined |
| Tree health field changed | That tree | Tree + combined |
| `workflow.job_created` | Job sites, assigned users | Workflow-aware rules (Teams) |

### 2.3 Time-based

| Trigger | Fan-out | Rule subset |
|---------|---------|-------------|
| Daily cron (org timezone) | Per-org batch windows | Seasonal, maintenance windows, digests |
| Season boundary (computed) | Region/species cohorts | Seasonal + combined |

**New:** “No activity in X days” requires **materialized last-activity** or query — document cost before shipping.

---

## 3. Execution placement (open)

Candidates:

- **Cloud Functions** — on schedule, on Firestore writes, on HTTP from ingestion
- **Cloud Run / worker** — heavier batch
- **App route** — only for pull; not primary for push

Document **latency targets** per trigger class when product sets SLOs (e.g. forecast update → first derived event &lt; N minutes).

---

## 4. Correlation and idempotency

- **`correlationId`:** one per fetch run or per nightly batch slice — groups many `intelligence.*` events for debugging.
- **`dedupeKey`:** per rule + subject + window — prevents duplicate user-visible alerts ([`derived-events-payload-schema.md`](./derived-events-payload-schema.md)).
- Writers should use **idempotent append** (same dedupe → no second visible alert) or **second event** that references the first — pick one policy and document in append service.

---

## 5. Tier consultation

Before evaluating:

1. Resolve **org / user** capabilities (subscription + entitlements).
2. Filter **rule catalog** to allowed `ruleId`s for that tier.
3. Filter **delivery** separately ([`../09-business/intelligence-tier-matrix.md`](../09-business/intelligence-tier-matrix.md)).

---

## 6. Related

- [`weather-ingestion-events.md`](./weather-ingestion-events.md)
- [`../../architecture/intelligence/FILE-STRUCTURE.md`](../../architecture/intelligence/FILE-STRUCTURE.md)
