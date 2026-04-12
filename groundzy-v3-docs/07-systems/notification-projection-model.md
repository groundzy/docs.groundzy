# Notification projection model (intelligence → inbox)

**Status:** Specification — evolves [`notification-system.md`](./notification-system.md) **as-built** toward **causal** notifications.  
**Depends on:** [`alert-notification-system.md`](./alert-notification-system.md), [`../05-data/derived-events-payload-schema.md`](../05-data/derived-events-payload-schema.md).

**See also:** [`notification-data-model-v3.md`](./notification-data-model-v3.md) (full v3 field list + actions), [`notification-channel-routing.md`](./notification-channel-routing.md) (severity → channels).

---

## 1. Principle

**Notifications are not the system of record** — they are a **projection** of intelligence / delivery decisions. The **audit trail** is `groundzy_events` (`intelligence.*`, optional `notification.*`). The Firestore `notifications` collection is **optimized for read** in the app.

---

## 2. Today (baseline)

See [`notification-system.md`](./notification-system.md): `AppNotification` with `type`, `title`, `message`, `actionUrl`, `read`.

**Gaps:** no `sourceEventId`, no `severity`, no channel lifecycle, weak typing on `type`.

---

## 3. Target shape (minimal viable upgrade)

Illustrative Firestore fields (names align with TypeScript in app when implemented):

| Field | Type | Purpose |
|-------|------|---------|
| `userId` | `string` | Recipient |
| `sourceEventId` | `string?` | **`groundzy_events` doc id** for the `intelligence.alert_triggered` (or `notification.created`) |
| `organizationId` | `string?` | Tenant scope; routing |
| `severity` | `enum` | `info` \| `recommendation` \| `warning` \| `critical` |
| `channel` | `enum` | `app` (default) \| `email` \| `sms` — for this row |
| `status` | `enum` | `pending` \| `sent` \| `delivered` \| `read` \| `failed` (subset may ship incrementally) |
| `type` | `string` | Closed union — see [`../../reference/notification-types.md`](../../reference/notification-types.md) |
| `title` | `string` | |
| `message` | `string` | |
| `actionUrl` or structured `payload` | | Prefer **typed payload** + router over free-form URLs long-term |
| `dedupeKey` | `string?` | Copy from intelligence payload for client-side merge |
| `createdAt` / `updatedAt` | timestamps | |

**Read path:** inbox can show **explainability** by fetching `sourceEventId` (when permissions allow) or embedding a short `evidenceSummary` on the notification doc.

---

## 4. Mapping from `intelligence.alert_triggered`

1. Append `intelligence.alert_triggered` to `groundzy_events` with full `evaluation` / `outputs`.
2. For each target user (see [`delivery-preferences-and-routing.md`](./delivery-preferences-and-routing.md)):
   - Create `notifications/{id}` with `sourceEventId` = that event’s id.
   - Set `channel` = `app` initially; email/SMS rows or provider jobs may be separate docs or subcollections.

**Idempotency:** same `dedupeKey` → same notification row (upsert) or no-op second write.

---

## 5. Security rules (direction)

- Client: **read** own notifications; **update** `read` / `status` where allowed.
- **Create** remains server-only (matches [`notification-system.md`](./notification-system.md)).

---

## 6. Related

- [`delivery-preferences-and-routing.md`](./delivery-preferences-and-routing.md)
- [`../../features/notification-center.md`](../../features/notification-center.md)
