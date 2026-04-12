# Derived events — payload schema (intelligence layer)

**Status:** Specification — align with implementation in `lib/groundzy/events/schema/` and append path.  
**Depends on:** [`event-system.md`](./event-system.md) §4, [`intelligence-rules.md`](./intelligence-rules.md), [`../07-systems/notification-projection-model.md`](../07-systems/notification-projection-model.md).

---

## 1. Purpose

Derived events are append-only records of **what the system concluded**. Payloads must support:

- **Explainability** — “why did I see this?”
- **Dedupe / idempotency** — stable keys, bounded re-emission
- **Tier and routing** — enough metadata for delivery without re-running rules

Envelope fields (`schemaVersion`, `id`, `type`, `organizationId`, `actorUserId`, `correlationId`, `causationEventId`, `subject`, `createdAtMs`) match the unified model in [`event-system.md`](./event-system.md). This doc defines **payload** bodies for `intelligence.*` (and optional `notification.*` derived records).

---

## 2. Shared payload fragments

These fragments appear inside multiple `type`-specific payloads.

### 2.1 `ruleRef`

| Field | Type | Description |
|-------|------|-------------|
| `ruleId` | `string` | Stable id (e.g. `wx.storm_weak_tree_v1`) |
| `version` | `number` | Monotonic rule version for audit |

### 2.2 `inputPointers`

Illustrative; refine per product. All optional; include what was actually used.

| Field | Type | Description |
|-------|------|-------------|
| `weatherSnapshotId` | `string?` | Id of stored forecast snapshot or ingestion event |
| `treeId` | `string?` | Tree subject |
| `propertyId` | `string?` | Property context |
| `workflowEntityType` | `string?` | e.g. `job` |
| `workflowEntityId` | `string?` | Job/quote id when relevant |

### 2.3 `evaluation`

| Field | Type | Description |
|-------|------|-------------|
| `conditionsMet` | `string[]` | Human- or machine-readable condition keys (e.g. `wind_gust_ge_50mph`) |
| `score` | `number?` | Optional numeric score |
| `inputsHash` | `string?` | Optional hash of normalized inputs for replay/debug |

### 2.4 `outputs` (alert-facing)

| Field | Type | Description |
|-------|------|-------------|
| `severity` | `"info" \| "recommendation" \| "warning" \| "critical"` | Maps to UI and default channel behavior |
| `message` | `string` | Primary user-facing explanation (may be templated) |
| `suggestedActions` | `string[]?` | Stable action keys: `open_tree`, `create_job`, `snooze`, `dismiss` |

### 2.5 Dedupe and lifecycle

| Field | Type | Description |
|-------|------|-------------|
| `dedupeKey` | `string` | **Required** for emitters that can retry. Stable across evaluations for same logical conclusion (e.g. `${ruleId}:${treeId}:${forecastWindowId}`) |
| `expiresAt` | `string?` | ISO-8601; alert no longer valid after (optional auto-resolve) |

---

## 3. Event types (normative names)

Exact strings must match [`../../reference/intelligence-event-types.md`](../../reference/intelligence-event-types.md) and the code enum when added.

### 3.1 `intelligence.rule_evaluated`

Emitted when a rule run completes **whether or not** it fired a user-visible alert (optional per product; can be sampled in production).

| Payload field | Type | Notes |
|---------------|------|--------|
| `ruleRef` | `ruleRef` | |
| `fired` | `boolean` | `true` if thresholds met |
| `inputPointers` | `inputPointers` | |
| `evaluation` | `evaluation` | |
| `dedupeKey` | `string` | Run-level or alert-level per policy |

### 3.2 `intelligence.alert_triggered`

Emitted when the system **surfaces** an alert (notification pipeline may follow).

| Payload field | Type | Notes |
|---------------|------|--------|
| `ruleRef` | `ruleRef` | |
| `inputPointers` | `inputPointers` | |
| `evaluation` | `evaluation` | |
| `outputs` | `outputs` | |
| `dedupeKey` | `string` | **Required** |
| `expiresAt` | `string?` | |

### 3.3 `intelligence.alert_resolved`

Emitted when an alert is **superseded**, **expires**, or **user dismisses** (policy-dependent).

| Payload field | Type | Notes |
|---------------|------|--------|
| `priorDedupeKey` or `priorEventId` | `string` | Links to original alert |
| `reason` | `string` | e.g. `expired`, `forecast_changed`, `user_dismissed` |
| `ruleRef` | `ruleRef` | |

### 3.4 `notification.created` (optional derived record)

If the delivery layer appends its own event (vs. only writing Firestore `notifications`), payload should include:

| Payload field | Type | Notes |
|---------------|------|--------|
| `intelligenceEventId` | `string` | Source derived event |
| `targetUserId` | `string` | Recipient |
| `channel` | `"app" \| "email" \| "sms"` | |
| `notificationId` | `string?` | Firestore doc id |

---

## 4. Storage

Same collection as domain events (`groundzy_events`) vs. parallel collection is an **implementation decision** ([`event-system.md`](./event-system.md) §4.5). Envelope shape stays consistent.

---

## 5. Related

- [`weather-ingestion-events.md`](./weather-ingestion-events.md) — snapshot ids for `weatherSnapshotId`
- [`evaluation-runtime.md`](./evaluation-runtime.md) — when evaluation runs and how correlation ids are set
- [`../07-systems/alert-notification-system.md`](../07-systems/alert-notification-system.md) — pipeline overview
