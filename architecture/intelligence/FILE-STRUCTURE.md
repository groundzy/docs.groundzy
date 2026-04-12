# Markdown file structure — Intelligence & derived events

This is the **full manifest** of documentation for the **derived event pipeline** (intelligence layer), **notifications as projections**, and **tiered evaluation/delivery**. Paths are relative to the **`docs` repository root** (`Groundzy/docs`).

**Legend**

| Tag | Meaning |
|-----|---------|
| **Exists** | File is present; use as source of truth |
| **Optional** | Extra splits if individual files grow too large |

---

## 1. Foundation — v3 data & events

| Path | Tag | Role |
|------|-----|------|
| `groundzy-v3-docs/05-data/data-model-overview.md` | Exists | Where intelligence fits in the larger data model |
| `groundzy-v3-docs/05-data/event-system.md` | Exists | **Primary vs derived**; §4 derived/computed events; causation, actors, dedupe; open decisions |
| `groundzy-v3-docs/05-data/intelligence-rules.md` | Exists | Rule families (weather, tree, combined), `ruleId`, severity, tier hints — **evaluation catalog** |
| `groundzy-v3-docs/05-data/derived-events-payload-schema.md` | Exists | Normative payloads for `intelligence.*`, explainability, `dedupeKey` |
| `groundzy-v3-docs/05-data/weather-ingestion-events.md` | Exists | Forecast snapshot ids, illustrative `weather.*` names |
| `groundzy-v3-docs/05-data/evaluation-runtime.md` | Exists | Targeted re-evaluation, triggers, tier filter, placement |
| `groundzy-v3-docs/05-data/data-flows.md` | Exists | Cross-feature flows |
| `groundzy-v3-docs/05-data/entities.md` | Exists | Entity boundaries |

---

## 2. Systems — alerts, notifications, delivery

| Path | Tag | Role |
|------|-----|------|
| `groundzy-v3-docs/07-systems/alert-notification-system.md` | Exists | **Pipeline overview** |
| `groundzy-v3-docs/07-systems/notification-system.md` | Exists | **As-built** in-app `notifications` |
| `groundzy-v3-docs/07-systems/notification-projection-model.md` | Exists | `sourceEventId`, severity, channel, status |
| `groundzy-v3-docs/07-systems/notification-data-model-v3.md` | Exists | v3 data model: fields, `actions`, per-channel rows |
| `groundzy-v3-docs/07-systems/notification-channel-routing.md` | Exists | Decision engine: severity × tier × prefs → channels |
| `groundzy-v3-docs/07-systems/delivery-preferences-and-routing.md` | Exists | Preferences, routing, digests, quiet hours |
| `groundzy-v3-docs/07-systems/README.md` | Exists | Systems index |
| `groundzy-v3-docs/08-integrations/email.md` | Exists | Resend; **§ Intelligence / digest (planned)** |
| `groundzy-v3-docs/08-integrations/sms-intelligence-delivery.md` | Exists | SMS channel spec (planned) |

---

## 3. Product — features & UX

| Path | Tag | Role |
|------|-----|------|
| `features/weather.md` | Exists | Pull recommendations, APIs |
| `features/intelligence-alerts.md` | Exists | Product overview for alerts |
| `features/notification-center.md` | Exists | Timeline UX |
| `features/profile-and-settings.md` | Exists | Inbox; extend for preferences |
| `groundzy-v3-docs/06-features/weather.md` | Exists | v3 product weather |
| `groundzy-v3-docs/06-features/trees.md` | Exists | Tree context |
| `groundzy-v3-docs/06-features/workflow.md` | Exists | Workflow hooks |

---

## 4. Business — tiers & monetization

| Path | Tag | Role |
|------|-----|------|
| `groundzy-v3-docs/01-product/tier-system.md` | Exists | Tier definitions |
| `groundzy-v3-docs/09-business/tier-rules.md` | Exists | Business rules |
| `groundzy-v3-docs/09-business/intelligence-tier-matrix.md` | Exists | Evaluation vs delivery levers |
| `architecture/tier-system-audit-2026-04-08.md` | Exists | Audit |
| `architecture/Groundzy v3 - Access & Permission System.md` | Exists | Access for routing |

---

## 5. Reference — schemas, APIs, Firestore

| Path | Tag | Role |
|------|-----|------|
| `reference/firestore-collections.md` | Exists | Extend when `notifications` / `groundzy_events` gain new fields |
| `reference/api-routes.md` | Exists | Weather APIs; extend for workers |
| `reference/intelligence-event-types.md` | Exists | **Canonical** event type catalog |
| `reference/notification-types.md` | Exists | **Canonical** notification `type` union |
| `groundzy-v3-docs/99-reference/glossary.md` | Exists | Terms |
| `groundzy-v3-docs/99-reference/vertical-slice-intelligence-v1.md` | Exists | First slice spec |
| `groundzy-v3-docs/99-reference/intelligence-event-types.md` | Exists | Mirror → `reference/` |
| `groundzy-v3-docs/99-reference/notification-types.md` | Exists | Mirror → `reference/` |

---

## 6. App-repo / lib — code-adjacent docs

| Path | Tag | Role |
|------|-----|------|
| `lib/weather/README.md` | Exists | Weather module file map |

---

## 7. Architecture hub

| Path | Tag | Role |
|------|-----|------|
| `architecture/intelligence/README.md` | Exists | Navigation hub |
| `architecture/intelligence/FILE-STRUCTURE.md` | Exists | This manifest |

---

## 8. Full tree (copy-paste)

```
docs/
├── architecture/
│   └── intelligence/
│       ├── README.md
│       └── FILE-STRUCTURE.md
├── features/
│   ├── weather.md
│   ├── intelligence-alerts.md
│   └── notification-center.md
├── groundzy-v3-docs/
│   ├── 05-data/
│   │   ├── event-system.md
│   │   ├── intelligence-rules.md
│   │   ├── derived-events-payload-schema.md
│   │   ├── evaluation-runtime.md
│   │   └── weather-ingestion-events.md
│   ├── 07-systems/
│   │   ├── alert-notification-system.md
│   │   ├── notification-system.md
│   │   ├── notification-projection-model.md
│   │   ├── notification-data-model-v3.md
│   │   ├── notification-channel-routing.md
│   │   └── delivery-preferences-and-routing.md
│   ├── 08-integrations/
│   │   ├── email.md
│   │   └── sms-intelligence-delivery.md
│   ├── 09-business/
│   │   └── intelligence-tier-matrix.md
│   └── 99-reference/
│       ├── glossary.md
│       ├── vertical-slice-intelligence-v1.md
│       ├── intelligence-event-types.md   # mirror
│       └── notification-types.md         # mirror
├── lib/
│   └── weather/
│       └── README.md
└── reference/
    ├── firestore-collections.md
    ├── api-routes.md
    ├── intelligence-event-types.md
    └── notification-types.md
```

---

## 9. Maintenance

When `GroundzyEventType` gains `intelligence.*` in code:

1. Update `reference/intelligence-event-types.md`.
2. Update `groundzy-v3-docs/05-data/derived-events-payload-schema.md` if payloads change.
3. Update `reference/firestore-collections.md` for `groundzy_events` and `notifications`.
