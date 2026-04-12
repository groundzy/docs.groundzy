# Intelligence Rules Layer

**Status:** Specification — evaluation logic catalog (rules only)  
**Scope:** What to evaluate and how conclusions are formed. **Not** delivery, channels, or routing — see [`../07-systems/alert-notification-system.md`](../07-systems/alert-notification-system.md).  
**Inputs:** Weather snapshots, tree/property state, time/season, optional workflow context.  
**Outputs:** Rule firings that map to trigger payloads (`severity`, `evidence`, `dedupeKey`, suggested actions) consumed by the alert engine.

---

## 1. Purpose

The **Intelligence Rules Layer** is the deterministic core (plus optional scoring/AI **assist**) that answers:

- Given **current inputs**, should we surface a conclusion?
- At what **severity**?
- With what **evidence** and **deduplication** key?

Rules are grouped into three families:

| Family | Primary signals | Typical use |
|--------|-----------------|-------------|
| **Weather** | Forecast and history for a location/property window | Planning, spray/water timing, generic hazard windows |
| **Tree** | Species, health, treatments, inspections, stored risk flags | Maintenance gaps, declining health, species/cadence checks |
| **Combined** | Weather × tree (and sometimes time/workflow) | Urgency when hazard and vulnerability overlap |

---

## 2. Rule record (conceptual)

Each rule should be identifiable and testable:

| Field | Description |
|-------|-------------|
| `ruleId` | Stable id (e.g. `wx.rain_spray_delay_48h`) |
| `family` | `weather` · `tree` · `combined` |
| `inputs` | Required data keys (forecast slice, tree health, season, etc.) |
| `condition` | Human-readable summary; implementation may be code or DSL |
| `defaultSeverity` | `INFO` · `RECOMMENDATION` · `WARNING` · `CRITICAL` |
| `dedupeStrategy` | e.g. per property + rule + day; per tree + rule + forecast run id |
| `tierHint` | Minimum capability / tier to **evaluate** this rule (product-defined) |
| `emitEvent` | Optional derived event name, e.g. `intelligence.weather.planning_hint` |

**Conflict handling:** If multiple rules fire for the same subject, the engine may **merge**, **take max severity**, or **prefer explicit ordering** per family — behavior should be documented per release.

---

## 3. Weather rules

**Triggers:** `weather.forecast.updated`, `weather.history.ingested`, scheduled re-evaluation, `weather.significant_change.detected` (when defined).

**Context:** Property or job site coordinates, timezone, evaluation window (e.g. next 24h / 48h / 7d).

| ruleId (example) | Condition (summary) | Default severity | Notes / evidence |
|------------------|------------------------|------------------|------------------|
| `wx.heavy_rain_spray_delay` | Precipitation above threshold in next 48h where spray/treatment would be ill-advised | `RECOMMENDATION` | Link forecast strip; not tree-specific |
| `wx.drought_stress_window` | Extended dry/hot period vs regional normals or thresholds | `WARNING` | May pair with tree rules in combined set |
| `wx.freeze_risk` | Air temp forecast below threshold in next N hours (season-dependent) | `WARNING` | INFO in mild bands if product prefers |
| `wx.high_wind` | Sustained gusts above threshold in window | `WARNING` | Relevant to limb/work zone safety generally |
| `wx.planning_window_favorable` | Mild, dry window suitable for field work | `INFO` | Positive scheduling hint |

**Parameters** (thresholds, windows) should be **configurable** per environment or region — avoid hard-coding in client-only code.

---

## 4. Tree rules

**Triggers:** `tree.health.updated`, `tree.treatment.recorded`, `tree.risk.assessed`, `schedule.evaluation.due`, CRUD on tree that affects health/treatment dates.

**Context:** Single tree (species, age class optional), health score/status, dates of last treatment/inspection, stored structural/pest risk where available.

| ruleId (example) | Condition (summary) | Default severity | Notes / evidence |
|------------------|------------------------|------------------|------------------|
| `tree.health_low` | Health score or status below configured threshold | `WARNING` | Snapshot of score + trend if present |
| `tree.inspection_overdue` | No inspection since before `max(days, season)` | `RECOMMENDATION` | Species may adjust cadence in config |
| `tree.treatment_gap` | Species (or tag) requires treatment on cadence; last treatment older than window | `RECOMMENDATION` | e.g. PHC, IPM — product taxonomy |
| `tree.pruning_window` | Season + species indicate typical pruning window open; no recent pruning record | `INFO` | Jurisdiction/species calendars in data, not only heuristics |
| `tree.stored_risk_elevated` | Structural or target-hazard flag set on tree | `WARNING` | Until inspected/mitigated |

---

## 5. Combined rules

**Triggers:** Any weather input **plus** tree/property context loaded in the same evaluation; optionally workflow (open job) to suppress duplicate work.

**Context:** Union of weather window + tree attributes + optional “open job exists for this tree.”

| ruleId (example) | Condition (summary) | Default severity | Notes / evidence |
|------------------|------------------------|------------------|------------------|
| `combo.storm_weak_tree` | High wind / storm signal **and** elevated structural risk or low health | `CRITICAL` | Explicit “seek professional assessment” copy where policy requires |
| `combo.rain_spray_scheduled` | Heavy rain in 48h **and** spray-type treatment scheduled or recently logged as planned | `RECOMMENDATION` | May integrate with workflow when present |
| `combo.drought_vulnerable` | Drought signal **and** species/stress profile indicating water sensitivity | `WARNING` | Evidence: drought metric + species tag |
| `combo.freeze_early_flush` | Freeze risk **and** species/region early flush / frost sensitivity | `WARNING` | Seasonal |
| `combo.open_job_duplicate` | Rule would fire but active job/quote already covers same concern | suppressed or `INFO` | Dedupe / workflow-aware branch |

Combined rules are the primary place for **moat-grade** differentiation: simple dashboards rarely correlate hazard with asset vulnerability.

---

## 6. Time and season without new weather events

Rules may run on **`calendar.season.transition`**, **`schedule.evaluation.due`**, or cron — e.g. “spring oak maintenance” — classified as **tree** or **combined** depending on whether weather inputs are required for that rule.

---

## 7. Severity and deduplication (rules perspective)

- **Severity** here is the rule’s **default**; the alert engine may downgrade per user prefs or tier (see alert system doc).
- **Dedupe:** weather-only rules often dedupe by **property + rule + day** or **forecast revision id**; tree rules by **tree + rule + time bucket**; combined rules may include **both** in the key to avoid double alerts from separate wx + tree runs.

---

## 8. Non-goals (this document)

- Channel selection, SMS, email, digests.
- User/org routing matrices.
- Canonical JSON Schema for events — lives with the v3 event catalog.

---

## 9. Related documents

- [`../07-systems/alert-notification-system.md`](../07-systems/alert-notification-system.md) — full pipeline, triggers, delivery, tiers, derived events.
