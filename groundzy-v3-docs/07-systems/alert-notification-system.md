# Alert & Notification System (Intelligence / Alert Engine)

**Status:** Specification — system layer not yet implemented  
**Depends on:** Event-first architecture (v3), weather, trees, health, workflow  
**Purpose:** Turn passive data into **system-driven awareness**: evaluate inputs, emit derived events, route notifications, and optionally drive workflow.

---

## 1. Problem statement

Individual subsystems exist (weather, trees, health, workflow), but there is no **observer** that:

- Monitors **relationships** across domains (e.g. forecast × tree risk × season).
- Runs **evaluation → trigger → delivery → action** consistently.
- Applies **tier**, **preference**, and **routing** rules.

Without this layer, the product **displays** information but does not **react**, **warn**, **suggest**, or **automate** in a unified way.

---

## 2. Architecture overview

Conceptual pipeline:

```
Inputs (existing data + domain events)
        ↓
Evaluation (rules + optional AI / scoring)
        ↓
Triggers (structured alert / intelligence events)
        ↓
Delivery (channels, frequency, tier gates)
        ↓
Surfaces (notification center, map, workflow hooks)
```

This sits **beside** raw domain events: domain events record *what happened*; **derived intelligence events** record *what the system concluded* and *what it decided to surface or automate*.

---

## 3. Event inputs

### 3.1 Weather

- **History** — past conditions relevant to treatment timing, stress, soil moisture proxies.
- **Forecast** — windows for risk (wind, freeze, heavy rain), scheduling (spray delay, watering).

Representative input events (names illustrative; align with canonical v3 event catalog when defined):

- `weather.forecast.updated`
- `weather.history.ingested` / `weather.significant_change.detected`

### 3.2 Tree state

- Species, age class, location context.
- Health score / status, trend if available.
- Last treatment / inspection timestamps.
- Risk indicators (structure, pests, proximity to targets).

Illustrative inputs:

- `tree.health.updated`
- `tree.treatment.recorded`
- `tree.risk.assessed`

### 3.3 Time & seasonality

- Season, month, growth cycle for region/species.
- Scheduled evaluations (cron / calendar), not only reactive pushes.

Illustrative:

- `calendar.season.transition`
- `schedule.evaluation.due` (per tree, per property, per org)

### 3.4 Workflow & commercial context

- Jobs, quotes, services, assignments — for routing (“who cares?”) and actions (“create job from alert”).

Illustrative:

- `workflow.job.status_changed`
- `workflow.quote.sent`

### 3.5 Cross-cutting requirement

All inputs must be **correlatable** in evaluation (stable IDs for user, org, tree, property, workflow entity). Evaluation is **stateless per run** but may read **materialized snapshots** (e.g. latest forecast + latest tree record) from stores or projections.

---

## 4. Rule engine (evaluation layer)

### 4.1 Responsibilities

1. **Subscribe** to relevant input events and/or periodic schedules.
2. **Load context** (tree, property, user/org, tier, preferences).
3. **Evaluate** deterministic rules and optional scored models.
4. **Emit** trigger payloads (see §5) and **derived events** (see §8).

### 4.2 Rule categories (examples)

| Category        | Example condition                                      | Example outcome                          |
|----------------|---------------------------------------------------------|------------------------------------------|
| Weather        | Heavy rain within 48h                                   | Delay spraying recommendation            |
| Weather        | Drought / heat stress window                          | Watering / inspection alert              |
| Tree health    | Low health score below threshold                      | Inspection or treatment suggestion     |
| Species × time | Oak + spring + no treatment in 12 months              | Risk / maintenance alert               |
| Combined       | Storm forecast + structurally weak tree               | Urgent risk alert                        |
| Seasonal       | Species-specific pruning window open                  | Informational / scheduling suggestion    |

### 4.3 Implementation shape (non-prescriptive)

- **Deterministic rules first** (maintainable, testable, explainable).
- **Optional AI / ML** for ranking, wording, or anomaly detection — outputs must still map to the same **alert types** and **delivery** contracts.
- **Idempotency:** re-evaluation of the same conditions should not spam; use deduplication keys (e.g. tree + alert kind + time bucket).

### 4.4 Outputs of evaluation

- Binary or graded **decision** (emit / suppress / downgrade).
- **Severity** and **alert type** (§5).
- **Suggested actions** (deep links, workflow templates) without forcing automation.

---

## 5. Alert types (triggers)

Structured **trigger** model (names indicative):

| Field | Description |
|-------|-------------|
| `alertType` | Category: recommendation, maintenance, risk, workflow, system |
| `severity` | `INFO` · `RECOMMENDATION` · `WARNING` · `CRITICAL` |
| `subject` | Tree, property, job, org — with IDs |
| `title` / `body` | User-facing copy (templated) |
| `evidence` | Pointers to inputs (forecast slice, health snapshot, rule id) |
| `actions` | Optional: open tree, create job, snooze, dismiss |
| `dedupeKey` | For suppression across repeated evaluations |

**Severity** guides UI prominence, map styling, and default channel behavior (see §6).

---

## 6. Delivery channels & preferences

### 6.1 Channels

| Channel | Role | Notes |
|---------|------|--------|
| **In-app** | Default; full rich actions | Notification center, badges, deep links |
| **Email** | Async, broader reach | Digest-capable; compliance with unsubscribe |
| **SMS** | High-urgency, paid tiers | Rate limits, consent, regulatory constraints |

### 6.2 User / org preferences

- **Per-channel** opt-in/out where allowed by tier.
- **Frequency:** instant, daily digest, weekly digest (per channel or global).
- **Quiet hours** (optional): suppress non-critical outside window; escalate only `CRITICAL` if policy allows.

### 6.3 Routing logic (“who gets notified?”)

- Property owners vs team members vs assigned pros — based on **role**, **sharing**, and **workflow participation**.
- Org-level defaults with **user overrides** where product policy allows.
- **Audit:** record who was targeted and why (minimal PII; sufficient for support).

### 6.4 Monetization alignment

SMS and advanced routing may be **tier-gated** (§7). Email digests might be gated partially (e.g. weekly digest only on higher tiers). Exact mapping is product/legal dependent; the **system** must support **capability flags** per tier, not hard-coded UI checks only.

---

## 7. Tier gating

Tiers control **which intelligence runs**, **which channels unlock**, and **automation depth**. Example matrix (product to confirm):

| Tier   | Intelligence scope | Channels | Automation / workflow |
|--------|--------------------|----------|------------------------|
| Home   | Basic weather + generic seasonal tips | In-app | Limited |
| Plus   | Smarter recommendations, more rules | In-app, email | Suggestions |
| Pro    | Tree-specific insights, richer combined rules | + email digests | Deeper actions |
| Teams  | Workflow-aware alerts, assignment routing | + SMS where enabled | Workflow triggers, team routing |

**Implementation note:** expose **capabilities** (e.g. `canUseSms`, `canRunRuleSet: 'pro_tree_risk'`) from subscription/billing layer; evaluation and delivery consult the same source of truth.

---

## 8. Derived intelligence events (event-first)

Illustrative chain:

1. `weather.forecast.updated`
2. `intelligence.tree_risk.detected` (derived)
3. `notification.created` (derived)
4. `workflow.job.suggested` or user-initiated `workflow.job.created_from_alert` (if implemented)

Domain events remain the **audit trail**; intelligence events are **first-class** for analytics, debugging, and “why did I get this?” in the UI.

---

## 9. User-facing outputs

### 9.1 Notification center

- Timeline of alerts with **severity**, **filter**, **read state**.
- **Action buttons** tied to `actions` on the trigger model.
- **Explainability:** link to evidence (forecast snippet, tree snapshot, rule label).

### 9.2 Map & tree views

- Optional **overlays** for risk / urgency by tree or zone.
- Consistent legend with severity colors.

### 9.3 Workflow integration

- “Create job from alert,” “Convert to quote” where tier and permissions allow.
- Idempotent creation to avoid duplicate jobs from repeated notifications.

---

## 10. Non-goals (initial phase)

- Replacing human judgment for safety-critical arborist decisions.
- Guaranteed delivery of SMS/email without provider SLAs and user consent flows.
- Full real-time streaming of every weather pixel — **batch + significant-change** triggers are acceptable.

---

## 11. Related work

- v3 **event system** — canonical event names, schemas, and writers.
- **Permissions** — who may receive org/property alerts.
- **Billing / tiers** — capability flags for SMS and advanced rule sets.
- [`../05-data/intelligence-rules.md`](../05-data/intelligence-rules.md) — weather, tree, and combined rule catalog.

---

## 12. Open decisions

- Canonical event names and JSON schemas for `intelligence.*` and `notification.*`.
- Where evaluation runs (Cloud Functions, scheduled jobs, queue workers) and latency targets.
- Retention policy for notifications and derived events.
- AI disclosure and safety copy for automated recommendations.
