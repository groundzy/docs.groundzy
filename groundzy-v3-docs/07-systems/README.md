# Cross-feature systems

These documents describe **shared foundations** that span multiple product features—not individual drawers or flows.

| Doc | System |
|-----|--------|
| [event-history-system.md](./event-history-system.md) | Activity, history, tree events, work_items, merges |
| [workflow-system.md](./workflow-system.md) | Request→invoice pipeline, settings, mirrors |
| [notification-system.md](./notification-system.md) | In-app notifications vs inbox (as-built) |
| [alert-notification-system.md](./alert-notification-system.md) | Intelligence pipeline: evaluation → derived events → delivery |
| [notification-projection-model.md](./notification-projection-model.md) | Notifications as projection of `intelligence.*` |
| [delivery-preferences-and-routing.md](./delivery-preferences-and-routing.md) | Who gets notified; channels; digests; quiet hours |
| [notification-data-model-v3.md](./notification-data-model-v3.md) | v3 Firestore + client shape: channels, severity, actions, provenance |
| [notification-channel-routing.md](./notification-channel-routing.md) | Decision engine: severity × tier × prefs → channel(s) |
| [permissions-system.md](./permissions-system.md) | Rules, tiers, roles, tree ACL |
| [search-system.md](./search-system.md) | Global search architecture |

**Goal:** Surface **duplication**, **fragmentation**, and **missing abstractions** so v3 can replace ad hoc wiring with reusable layers (`Groundzy v3/00-foundation/principles.md`).
