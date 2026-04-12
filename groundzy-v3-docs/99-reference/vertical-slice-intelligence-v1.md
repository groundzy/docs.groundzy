# Vertical slice v1 — storm × weak tree (CRITICAL)

**Status:** Living spec for the **first** shipped intelligence pipeline — update when implemented.

**Goal:** Prove **derived events + dedupe + one in-app notification** before generalizing.

---

## Scenario

When **forecast** indicates high wind / storm risk **and** a **tree** has elevated structural risk or poor health, emit a **CRITICAL** alert (tier and product permitting).

---

## Rule sketch

| Field | Value |
|-------|--------|
| `ruleId` | `combined.storm_weak_tree_v1` (example) |
| `version` | `1` |

**Inputs:**

- Normalized forecast (reuse `computeTreeImpact` / recommendation storm paths or shared thresholds from `lib/weather/tree-impact.ts`).
- Tree: `health.overallStatus`, `health.riskLevel` — see [`../05-data/intelligence-rules.md`](../05-data/intelligence-rules.md).

**Dedupe key (example):**

```
dedupeKey = `${ruleId}:${treeId}:${forecastWindowId}`
```

Where `forecastWindowId` is stable for the current provider fetch (e.g. tied to [`weather-ingestion-events.md`](../05-data/weather-ingestion-events.md) snapshot).

---

## Events emitted

1. `intelligence.alert_triggered` with full [`derived-events-payload-schema.md`](../05-data/derived-events-payload-schema.md) payload.
2. One `notifications` doc per user with `sourceEventId`, `severity: critical`, `channel: app`.

---

## Test scenarios

| # | Case | Expected |
|---|------|----------|
| 1 | Storm + high-risk tree | Single alert; dedupe stable on retry |
| 2 | Storm + healthy tree | No alert or lower severity per rule |
| 3 | No storm + weak tree | No combined alert (may be separate tree rule) |
| 4 | Same conditions re-evaluated within window | No duplicate (idempotent writer) |

---

## Rollback

- Disable rule flag in config **or** stop evaluator subscription — notifications cease; past `groundzy_events` remain audit.

---

## Related

- [`../05-data/evaluation-runtime.md`](../05-data/evaluation-runtime.md)
- [`../../architecture/intelligence/README.md`](../../architecture/intelligence/README.md)
