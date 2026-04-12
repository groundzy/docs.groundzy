# Intelligence & derived events (documentation hub)

This folder holds the **navigation index** for Groundzy’s **second pipeline**: state and signals → evaluation → **derived (`intelligence.*`) events** → side effects (notifications, workflow suggestions).

**Implementation plan (engineering):** [`../intelligence-implementation-plan.md`](../intelligence-implementation-plan.md) — phased build, insertion points, risks, grounded in `app.groundzy`.

**Canonical deep specs** for v3 live under [`groundzy-v3-docs/`](../../groundzy-v3-docs/README.md). This hub does not duplicate them; it tells you **what to read** and **what is still missing**.

## Start here

| Goal | Document |
|------|----------|
| Primary vs derived pipelines, derived event definition | [`groundzy-v3-docs/05-data/event-system.md`](../groundzy-v3-docs/05-data/event-system.md) §3–4 |
| Rule catalog, severities, `ruleId` / dedupe strategy | [`groundzy-v3-docs/05-data/intelligence-rules.md`](../groundzy-v3-docs/05-data/intelligence-rules.md) |
| End-to-end alert engine (evaluation → delivery → UI) | [`groundzy-v3-docs/07-systems/alert-notification-system.md`](../groundzy-v3-docs/07-systems/alert-notification-system.md) |
| Today’s in-app notifications (Firestore, gaps) | [`groundzy-v3-docs/07-systems/notification-system.md`](../groundzy-v3-docs/07-systems/notification-system.md) |
| Pull-based weather + pure functions (`computeTreeImpact`, `computeRecommendations`) | [`features/weather.md`](../../features/weather.md), [`lib/weather/README.md`](../../lib/weather/README.md) |
| Workflow / CRM event storage patterns | [`architecture/canonical-workflow-schema-v3.md`](../canonical-workflow-schema-v3.md) |
| Payload shapes for `intelligence.*` | [`groundzy-v3-docs/05-data/derived-events-payload-schema.md`](../groundzy-v3-docs/05-data/derived-events-payload-schema.md) |
| Evaluation triggers & runtime | [`groundzy-v3-docs/05-data/evaluation-runtime.md`](../groundzy-v3-docs/05-data/evaluation-runtime.md) |
| Notifications as projections | [`groundzy-v3-docs/07-systems/notification-projection-model.md`](../groundzy-v3-docs/07-systems/notification-projection-model.md) |
| Tier matrix (rules vs channels) | [`groundzy-v3-docs/09-business/intelligence-tier-matrix.md`](../groundzy-v3-docs/09-business/intelligence-tier-matrix.md) |
| Product: alerts & inbox | [`features/intelligence-alerts.md`](../features/intelligence-alerts.md), [`features/notification-center.md`](../features/notification-center.md) |
| Event type catalog (canonical) | [`reference/intelligence-event-types.md`](../reference/intelligence-event-types.md) |

## Full file list

See **[FILE-STRUCTURE.md](./FILE-STRUCTURE.md)** for the complete tree.

## Code anchors (implementation)

- `lib/groundzy/events/schema/system.ts` — `GroundzyEventType` (today: `workflow.*`, `tree.*` only)
- `lib/weather/tree-impact.ts`, `lib/weather/recommendations.ts` — pure evaluation (today: mostly **pull** via API)

Keep **[FILE-STRUCTURE.md](./FILE-STRUCTURE.md)** and [`reference/intelligence-event-types.md`](../reference/intelligence-event-types.md) aligned when types ship in code.
