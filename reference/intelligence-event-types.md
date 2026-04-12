# Intelligence & weather ingestion — event type catalog

**Status:** **Authoritative enum** for docs; **must stay in sync** with `lib/groundzy/events/schema/system.ts` (`GroundzyEventType`) when types land in code.

---

## 1. Primary pipeline (exists in code)

Workflow and tree timeline — see app `system.ts`:

- `workflow.request_created`, `workflow.quote_*`, `workflow.job_*`, `workflow.invoice_*`
- `tree.note_*`, `tree.service_*`, `tree.measurement_*`, `tree.inspection_*`, `tree.timeline_entry_removed`

---

## 2. Derived — intelligence

Normative payload shapes: [`../groundzy-v3-docs/05-data/derived-events-payload-schema.md`](../groundzy-v3-docs/05-data/derived-events-payload-schema.md).

**Implemented in code:** `intelligence.alert_triggered` — see `lib/groundzy/events/schema/intelligence.ts`, `lib/groundzy/server/append-intelligence-event.ts`.

| Type | Description |
|------|-------------|
| `intelligence.rule_evaluated` | Rule run completed (may or may not surface UI) — not yet emitted |
| `intelligence.alert_triggered` | User-visible alert conclusion |
| `intelligence.alert_resolved` | Expired, superseded, or dismissed — not yet emitted |

**Naming:** use **dot** separators; prefix `intelligence.` for all derived interpretation events.

---

## 3. Weather ingestion (illustrative)

Stored or logged when push evaluation exists — [`../groundzy-v3-docs/05-data/weather-ingestion-events.md`](../groundzy-v3-docs/05-data/weather-ingestion-events.md).

| Type | Description |
|------|-------------|
| `weather.forecast.updated` | New snapshot for keyed region/window |
| `weather.current.updated` | Current conditions refresh (if modeled) |
| `weather.significant_change.detected` | Threshold-based re-evaluation hook |

These may be **internal** only until promoted to `groundzy_events`.

---

## 4. Delivery (optional derived)

| Type | Description |
|------|-------------|
| `notification.created` | Delivery layer committed channel-specific notification |

---

## 5. Workflow suggestions (optional)

| Type | Description |
|------|-------------|
| `workflow.job_suggested` | System recommends job — if implemented |

---

## 6. Maintenance

On any change:

1. Update `GroundzyEventType` in app repo.
2. Update this file.
3. Update [`../groundzy-v3-docs/05-data/event-system.md`](../groundzy-v3-docs/05-data/event-system.md) if semantics change.
