# Intelligence Rules Layer

**Step 2 — Define the rules catalog (evaluation only)**  
**Status:** Specification — evaluation logic and rule families. **Not** delivery, channels, routing, or storage schemas — see [`../07-systems/alert-notification-system.md`](../07-systems/alert-notification-system.md).  
**Inputs:** Weather snapshots, tree/property state, **time / season / growth windows**, optional workflow context.  
**Outputs:** Rule firings that map to trigger payloads (`severity`, `evidence`, `dedupeKey`, suggested actions) consumed by the Alert / Intelligence engine.

---

## 1. Purpose

The **Intelligence Rules Layer** is the deterministic core (plus optional scoring or AI **assist**) that answers:

- Given **current inputs**, should we surface a conclusion?
- At what **severity**?
- With what **evidence** and **deduplication** key?

Rules are grouped into **four** topical areas; three are **families** that produce firings, and **time/season** is **cross-cutting context** required by many rules:

| Area | Role |
|------|------|
| **Time & season** | Shared context (hemisphere, month, dormant vs growing season, local frost windows) — not a separate “family” but **required inputs** for weather, tree, and combined rules |
| **Weather** | Forecast and history for a location/property window |
| **Tree** | Species, health, treatments, inspections, stored risk |
| **Combined** | Weather × tree (and sometimes time/workflow) — urgency when hazard and vulnerability overlap |

---

## 2. Shared inputs (evaluation context)

Before any rule runs, the engine should assemble a **evaluation context** (names illustrative):

| Input key | Source (conceptual) | Used by |
|-----------|---------------------|---------|
| `coordinates` + `timezone` | Property / job site | Weather, local time, season |
| `forecastWindow` | Next 24h / 48h / 7d slices | Weather, combined |
| `weatherHistoryRecent` | Recent history or anomalies | Weather, drought/stress proxies |
| `calendar` | Local date/time | All families |
| `hemisphere` / `hardinessZone` (optional) | Property or org defaults | Season, freeze, pruning windows |
| `tree.species`, `tree.health`, `tree.lastTreatment`, `tree.lastInspection`, `tree.riskFlags` | Tree documents / projections | Tree, combined |
| `workflow.openJobsNearby` (optional) | Work items | Combined (crew/schedule conflict hints) |

**Time & season** rules do not stand alone: they **qualify** other rules (e.g. “pruning window” depends on species + hemisphere + dormant season, not just the calendar).

---

## 3. Rule record (conceptual)

Each rule should be identifiable, testable, and versionable:

| Field | Description |
|-------|-------------|
| `ruleId` | Stable id (e.g. `wx.heavy_rain_spray_delay_48h`) |
| `family` | `weather` · `tree` · `combined` |
| `inputs` | Required data keys from the evaluation context |
| `condition` | Human-readable summary; implementation may be code or DSL |
| `defaultSeverity` | `INFO` · `RECOMMENDATION` · `WARNING` · `CRITICAL` |
| `dedupeStrategy` | e.g. per property + rule + day; per tree + rule + forecast run id |
| `tierHint` | Minimum capability / tier to **evaluate** this rule (product-defined) |
| `emitEvent` | Optional derived event name, e.g. `intelligence.weather.planning_hint` |

**Conflict handling:** If multiple rules fire for the same subject, the engine may **merge**, **take max severity**, or **prefer explicit ordering** per family — behavior must be documented per release.

**Parameters** (thresholds, windows, °C vs °F) should be **configurable** per environment or region — avoid hard-coding only in client-only code.

---

## 4. Time, season, and growth context

Use this layer to **gate** or **adjust** severity for tree and combined rules.

| Concept | Role in rules |
|---------|---------------|
| **Local date & timezone** | Digest boundaries, “today,” frost date comparisons |
| **Meteorological vs astronomical season** | Product choice; document which definition is used |
| **Dormant vs growing season** | Species-dependent pruning and some treatment timing |
| **Growth flush / pest pressure windows** (region-specific) | Optional `ruleId` hooks when data exists |
| **Day-of-week** (optional) | Scheduling hints for pros (e.g. avoid Monday overload) — low priority |

**Illustrative time-only signals** (usually combined with tree or weather rules, not standalone products):

| ruleId (example) | Condition (summary) | Notes |
|------------------|----------------------|-------|
| `time.frost_window_approaching` | Calendar + regional typical first-frost band | Pairs with cold forecasts for combined severity |
| `time.spring_inspection_cadence` | Spring start + species with annual inspection expectation | Tree family when `lastInspection` stale |

---

## 5. Weather rules

**Triggers:** `weather.forecast.updated`, `weather.history.ingested`, scheduled re-evaluation, `weather.significant_change.detected` (when defined).

**Context:** Property or job site coordinates, timezone, evaluation window (e.g. next 24h / 48h / 7d).

| ruleId (example) | Condition (summary) | Default severity | Notes / evidence |
|------------------|---------------------|------------------|------------------|
| `wx.heavy_rain_spray_delay` | Precipitation above threshold in next 48h where spray/treatment would be ill-advised | `RECOMMENDATION` | Link forecast strip; not tree-specific |
| `wx.drought_stress_window` | Extended dry/hot period vs regional normals or thresholds | `WARNING` | Often pairs with tree rules in combined set |
| `wx.freeze_risk` | Air temp forecast below threshold in next N hours (season-dependent) | `WARNING` | `INFO` in mild bands if product prefers softer UX |
| `wx.high_wind` | Sustained gusts above threshold in window | `WARNING` | General work-zone and limb safety |
| `wx.planning_window_favorable` | Mild, dry window suitable for field work | `INFO` | Positive scheduling hint |
| `wx.severe_alert_active` | Provider or NWS-style alert polygon overlaps property | `WARNING` or `CRITICAL` | Map to product’s alert ingestion model |

---

## 6. Tree rules

**Triggers:** `tree.health.updated`, `tree.treatment.recorded`, `tree.risk.assessed`, `schedule.evaluation.due`, CRUD on tree that affects health/treatment dates.

**Context:** Single tree (species, age class optional), health score/status, dates of last treatment/inspection, stored structural/pest risk where available.

| ruleId (example) | Condition (summary) | Default severity | Notes / evidence |
|------------------|---------------------|------------------|------------------|
| `tree.health_low` | Health score or status below threshold | `WARNING` | Trend down may bump severity |
| `tree.treatment_overdue` | Species- or service-specific max interval since last treatment exceeded | `RECOMMENDATION` | Parameters per species/service catalog |
| `tree.inspection_overdue` | No inspection within policy interval | `RECOMMENDATION` | May be law/contract sensitive — copy review |
| `tree.risk_elevated` | Stored structural/pest/target risk above threshold | `WARNING` | Foundation for combined storm rules |
| `tree.pruning_window_open` | Species + dormant season + no recent structural prune | `INFO` | Requires time/season context |
| `tree.new_planting_followup` | Recent planting + no check-in measurement/visit | `INFO` | Optional; data-dependent |

---

## 7. Combined rules

**Triggers:** Same as weather and tree, plus **scheduled fusion** runs that load both forecast slices and tree sets for a property (e.g. nightly or on forecast refresh).

**Context:** Forecast window + per-tree vulnerability (health, risk, species) + season.

| ruleId (example) | Condition (summary) | Default severity | Notes / evidence |
|------------------|---------------------|------------------|------------------|
| `combo.storm_weak_tree` | High wind or ice forecast **and** tree risk/health over threshold | `CRITICAL` or `WARNING` | Classic moat rule — explainability critical |
| `combo.freeze_sensitive_species` | Freeze risk **and** species/condition known to be sensitive | `WARNING` | May reference hardiness vs forecast min |
| `combo.drought_stressed_canopy` | Drought window **and** low health or water-stress indicators | `WARNING` | Two-signal confirmation |
| `combo.heavy_rain_recent_treatment` | Heavy rain in 48h **and** recent spray/treatment that rain undermines | `RECOMMENDATION` | Needs treatment timestamp + product semantics |
| `combo.job_weather_conflict` | Open job scheduled in window with hazardous weather | `WARNING` | Teams / workflow-aware; optional tier |

---

## 8. Severity guidelines

| Level | User expectation |
|-------|------------------|
| `INFO` | Awareness; no immediate action required |
| `RECOMMENDATION` | Action suggested; user can dismiss |
| `WARNING` | Risk or opportunity requires attention soon |
| `CRITICAL` | Imminent safety or asset risk; stricter dedupe and routing policies |

---

## 9. Non-goals (this document)

- Channel selection, SMS/email/app routing, or user preferences.
- JSON Schema for payloads — belongs with the event catalog and alert engine.
- Guaranteed accuracy of weather or arboricultural advice — rules are **decision support**, not professional certification.

---

## 10. Related documents

- [`../07-systems/alert-notification-system.md`](../07-systems/alert-notification-system.md) — pipeline, delivery, tiers, notification center.
- [`event-system.md`](./event-system.md) — unified events, derived events, dedupe alignment.
- v3 **event catalog** — canonical names for `weather.*`, `tree.*`, `intelligence.*` (when published).
- **Entities / tree model** — fields that feed `inputs` above.

---

## 11. Open decisions

- DSL vs code-only rules; how non-engineers adjust thresholds.
- Regional defaults pack (JSON) vs single global defaults.
- How AI assist **augments** (score bump, narrative) without breaking deterministic `ruleId` auditability.
