# Intelligence & alerts (product overview)

**Nature:** System-driven **recommendations, warnings, and critical alerts** derived from weather, tree state, time, and (on higher tiers) workflow context. **Not** the same as team **messaging** (`conversations`).

---

## What users experience

- **Proactive guidance** — e.g. storm risk + vulnerable tree, watering skip when rain is coming, inspection suggestions when health or risk thresholds are met.
- **Explainability** — alerts should answer **why** (forecast snippet, rule label, tree state) when the notification center and intelligence layer are fully wired ([`notification-center.md`](./notification-center.md)).
- **Actions** — open tree, snooze, dismiss, and (where tier and permissions allow) **create job / quote** from an alert.

---

## Relationship to weather

- **Today:** Weather drawer and APIs deliver **pull** recommendations (`computeRecommendations`, `computeTreeImpact`) — see [`weather.md`](./weather.md).
- **Future:** Same pure functions feed **push** evaluation → `intelligence.*` events → inbox — see [`../groundzy-v3-docs/07-systems/alert-notification-system.md`](../groundzy-v3-docs/07-systems/alert-notification-system.md).

---

## Entry points (direction)

| Surface | Role |
|---------|------|
| **Inbox / Activity** | Badges combine messages + notifications today; intelligence rows gain `severity` and `sourceEventId` |
| **Map** | Optional risk/urgency overlays by tree |
| **Tree / workflow** | Deep links from alerts; “create job from alert” when enabled |

---

## Related docs

| Doc | Topic |
|-----|--------|
| [`../groundzy-v3-docs/05-data/intelligence-rules.md`](../groundzy-v3-docs/05-data/intelligence-rules.md) | Rule families |
| [`../groundzy-v3-docs/09-business/intelligence-tier-matrix.md`](../groundzy-v3-docs/09-business/intelligence-tier-matrix.md) | Tier vs rules vs channels |
| [`notification-center.md`](./notification-center.md) | Timeline UX |
| [`profile-and-settings.md`](./profile-and-settings.md) | Preferences when shipped |
