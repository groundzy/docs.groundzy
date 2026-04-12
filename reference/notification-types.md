# In-app notification `type` catalog

**Status:** Evolve from **free-form string** toward **closed union** — see [`../groundzy-v3-docs/07-systems/notification-system.md`](../groundzy-v3-docs/07-systems/notification-system.md).

**Client type today:** `AppNotification.type: string` (`lib/firebase/notifications.ts`).

---

## 1. Principles

- **`type`** is a **short discriminator** for UI (icon, color, routing), not the full story.
- **Payload** (future): structured fields for ids (`treeId`, `jobId`, `sourceEventId`) instead of parsing `actionUrl`.
- **Intelligence:** prefer `intelligence.*` + `sourceEventId` on the doc over inventing many one-off types.

---

## 2. Existing / informal (as surveyed)

| `type` (examples) | Source |
|---------------------|--------|
| Billing / Stripe | Webhooks, subscription events |
| Sharing / team | Invites, access changes |
| Workflow (ad hoc) | Server routes when present |

Document concrete values in code when auditing producers.

---

## 3. Target union (illustrative)

| `type` | Description |
|--------|-------------|
| `billing` | Subscription, payment, invoice |
| `sharing` | Tree/org share |
| `workflow` | Quote/job lifecycle (participant-safe links) |
| `intelligence` | Weather/tree/combined alerts — **subkind** in payload or `severity` on doc |

---

## 4. Intelligence sub-types (optional)

If a flat `intelligence` is too coarse, use **namespaced** strings:

- `intelligence.weather`
- `intelligence.tree_risk`
- `intelligence.workflow`

Keep **low cardinality** — avoid one type per marketing campaign.

---

## 5. Related

- [`../groundzy-v3-docs/07-systems/notification-projection-model.md`](../groundzy-v3-docs/07-systems/notification-projection-model.md)
- [`../groundzy-v3-docs/07-systems/notification-data-model-v3.md`](../groundzy-v3-docs/07-systems/notification-data-model-v3.md) — full v3 field list, `actions`, channels
- [`../groundzy-v3-docs/07-systems/notification-channel-routing.md`](../groundzy-v3-docs/07-systems/notification-channel-routing.md) — severity → channel matrix
- [`firestore-collections.md`](./firestore-collections.md) § notifications
