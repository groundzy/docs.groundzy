# Weather ingestion events & snapshots

**Status:** Specification — canonical names for **non-domain** weather inputs that trigger or correlate derived intelligence.  
**Depends on:** [`event-system.md`](./event-system.md), [`derived-events-payload-schema.md`](./derived-events-payload-schema.md).

---

## 1. Problem

Forecast data today is fetched **on demand** (`/api/weather/timeline`, etc.). For **push** evaluation you need stable references: **what forecast** did the rule use?

This doc defines **ingestion identifiers** and **illustrative event names** so `weatherSnapshotId` and `causationEventId` are meaningful.

---

## 2. Concepts

| Concept | Definition |
|---------|------------|
| **Fetch run** | One server-side call to the weather provider for a bounded spacetime (e.g. H3 cell + time window) |
| **Snapshot** | Normalized payload (aligned with `lib/weather/types.ts`) stored or hashed for dedupe |
| **Significant change** | Heuristic: e.g. alert added, wind gust step-change, freeze boundary crossed — triggers targeted re-evaluation |

---

## 3. Illustrative event names (catalog)

These may be stored as `groundzy_events` with a dedicated prefix, or only as **internal** job metadata until promoted. Align with [`reference/intelligence-event-types.md`](../../reference/intelligence-event-types.md) when finalized.

| Name | When |
|------|------|
| `weather.forecast.updated` | New snapshot available for a keyed region/window |
| `weather.current.updated` | Current conditions refresh (if modeled separately) |
| `weather.significant_change.detected` | Threshold-based; fan-out to combined rules |

**Note:** Provider webhooks are not assumed; **polling + cache** (existing H3 cache) may produce the same logical events internally.

---

## 4. Snapshot identity (`weatherSnapshotId`)

Recommended components (encode as opaque id or document id):

| Component | Example |
|-----------|---------|
| Spatial key | H3 index at agreed resolution (see app `lib/weather/cache.ts`) |
| Time window | Start/end of forecast slice or “run as of” timestamp |
| Provider | `visual_crossing` (or enum) |
| Payload hash | Optional — detect duplicate fetches |

**Correlation:** Intelligence runs triggered by the same fetch should share **`correlationId`** on derived events.

---

## 5. Relationship to pull path

`computeTreeImpact` / `computeRecommendations` remain **pure functions**. In push mode:

1. Ingestion or API path builds **normalized** timeline DTO.
2. Snapshot id is attached to **`intelligence.*`** payloads via `inputPointers.weatherSnapshotId`.

No rewrite of pure functions — only **call site** and **persistence** change.

---

## 6. Related

- [`features/weather.md`](../../features/weather.md) — product and API surface
- [`evaluation-runtime.md`](./evaluation-runtime.md) — fan-out from region to trees
