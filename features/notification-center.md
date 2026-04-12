# Notification center (timeline UX)

**Status:** Product + engineering — partial UI exists via **Activity** tab and inbox badges; **full** intelligence timeline is not yet implemented.

---

## Goals

- **Single place** to see system-generated alerts with **severity** (`info` → `critical`).
- **Filters** — by severity, by tree/property, read/unread, time range.
- **Explainability** — short “why” + link to evidence (forecast, tree snapshot) via `sourceEventId` when available ([`../groundzy-v3-docs/07-systems/notification-projection-model.md`](../groundzy-v3-docs/07-systems/notification-projection-model.md)).
- **Actions** — consistent buttons: open target, snooze, dismiss, create job/quote (tier-gated).

---

## Non-goals (initial)

- Replacing **messages** (`conversations`) — different system.
- Full **tree Activity** history — that remains tree timeline / v3 events; notification center is **alert-oriented**.

---

## Implementation notes

- **Data:** Firestore `notifications` evolves toward projection model; subscribe by `userId`, order by `createdAt`.
- **Read state:** `status` / `read` — align with [`../groundzy-v3-docs/07-systems/notification-system.md`](../groundzy-v3-docs/07-systems/notification-system.md).
- **Deep links:** prefer typed **payload** + router over ad hoc `actionUrl` strings long-term.

---

## Related

- [`intelligence-alerts.md`](./intelligence-alerts.md)
- [`../groundzy-v3-docs/07-systems/delivery-preferences-and-routing.md`](../groundzy-v3-docs/07-systems/delivery-preferences-and-routing.md)
