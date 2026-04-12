# Notification data model (Groundzy v3–ready)

**Status:** Specification — target model for Firestore + client types; **not** fully implemented in `app.groundzy` today.  
**Related:** [`notification-projection-model.md`](./notification-projection-model.md) (projection principle), [`../../reference/notification-types.md`](../../reference/notification-types.md) (type union), [`../05-data/derived-events-payload-schema.md`](../05-data/derived-events-payload-schema.md) (intelligence payloads).

---

## 1. Design principle

**Intelligence events are source of truth for “what happened and why.”**  
**Notifications are deliverables** — one or more **per recipient × channel** rows (or provider jobs) derived from a single `intelligence.alert_triggered` (or workflow domain event).

**Actionability:** If a notification does not support a **clear next step** (open tree, open job, create quote, view risk), treat it as low-value noise — prefer suppressing at the **decision layer** ([`notification-channel-routing.md`](./notification-channel-routing.md)).

---

## 2. Channels (tier 1)

| Channel | Role | Implementation note (today) |
|---------|------|-------------------------------|
| **In-app** | Primary UI: notification center, badges, drawers | `notifications` collection + client module `lib/firebase/notifications.ts` in **app** repo |
| **Email** | Async, digest, stakeholders | [`../08-integrations/email.md`](../08-integrations/email.md) — Resend |
| **SMS** | Urgent, interrupt-only | Spec only — [`../08-integrations/sms-intelligence-delivery.md`](../08-integrations/sms-intelligence-delivery.md) |

**Not equal:** Same logical alert may produce **one in-app doc** + **optional** email job + **optional** SMS — see routing doc.

**Later (optional):** Browser push (permission-based), workflow-native toasts — out of core v3 data model.

---

## 3. Severity (drives routing)

Normative enum aligned with intelligence outputs ([`derived-events-payload-schema.md`](../05-data/derived-events-payload-schema.md)):

| Severity | Meaning (product) |
|----------|-------------------|
| `info` | FYI; low urgency |
| `recommendation` | Suggested action, not urgent |
| `warning` | Should address; elevated attention |
| `critical` | Time-sensitive or safety-adjacent; may use interrupt channels |

Default **channel eligibility** by severity is defined in [`notification-channel-routing.md`](./notification-channel-routing.md) (not duplicated here to avoid drift).

---

## 4. Firestore: `notifications/{notificationId}`

### 4.1 Core fields (all channels represented as rows or linked jobs)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `userId` | `string` | Yes | Recipient |
| `organizationId` | `string?` | No | Tenant scope; indexing and ACL |
| `channel` | `"app"` \| `"email"` \| `"sms"` | Yes | Which delivery this row represents |
| `severity` | `info` \| `recommendation` \| `warning` \| `critical` | Yes | Copied from intelligence `outputs.severity` |
| `type` | `string` | Yes | Closed union — see [`reference/notification-types.md`](../../reference/notification-types.md) (e.g. `intelligence`, `workflow`, `billing`) |
| `title` | `string` | Yes | Short headline |
| `message` | `string` | Yes | Body; keep SMS-derived copies short at sender |
| `read` | `boolean` | Yes | In-app read state; for email/SMS may be N/A or mirror “ack” |
| `createdAt` | `Timestamp` | Yes | |
| `updatedAt` | `Timestamp?` | No | |

### 4.2 Provenance & dedupe

| Field | Type | Description |
|-------|------|-------------|
| `sourceEventId` | `string?` | `groundzy_events` document id (`intelligence.alert_triggered` or domain event) |
| `dedupeKey` | `string?` | Same logical alert as intelligence payload — prevents duplicate rows on retry |
| `idempotencyKey` | `string?` | Optional writer-side key for email/SMS dispatch |

### 4.3 Delivery lifecycle (especially email/SMS)

| Field | Type | Description |
|-------|------|-------------|
| `status` | `pending` \| `sent` \| `delivered` \| `failed` \| `read` | Per channel; app rows may use `read` for inbox |
| `providerMessageId` | `string?` | Resend / SMS provider id for support |

### 4.4 Actionability (prefer structured over opaque URLs)

Prefer a **small payload object** (Firestore map) so the client router does not parse strings:

| Field | Type | Description |
|-------|------|-------------|
| `actions` | `array` of `{ kind, labelKey?, treeId?, jobId?, quoteId?, route? }` | **kind**: `open_tree` \| `open_job` \| `open_quote` \| `create_quote` \| `view_risk` \| `dismiss` |
| `actionUrl` | `string?` | **Legacy / deep link** — optional fallback while migrating |

Example conceptual payload:

```json
{
  "kind": "open_tree",
  "treeId": "t_abc123",
  "route": "/drawers/view-tree?treeId=t_abc123"
}
```

**Rule:** Every **intelligence**-originated in-app notification should include **at least one** `actions[]` entry or a documented exception (pure info).

---

## 5. Alternate shapes (implementation options)

**Option A — One document per channel (recommended for clarity):**  
Same logical alert → up to three docs (`channel`: app, email, sms) sharing `sourceEventId` + `dedupeKey`.

**Option B — Single doc with subcollection `deliveries/{channel}`:**  
Heavier client; better if one row must aggregate status.

**Option C — In-app only in `notifications`; email/SMS in queue collection:**  
e.g. `notification_outbox` — use when provider retries matter.

Choose one per implementation phase; **Option A** matches the simplest queries for inbox (`where channel == app`).

---

## 6. Email-specific (optional fields or separate collection)

| Field | Purpose |
|-------|---------|
| `emailTemplateId` | Which Resend template |
| `digestBatchId` | Groups rows into one sent email |
| `unsubscribeToken` | Compliance |

---

## 7. Client type (`AppNotification`)

Evolve `lib/firebase/notifications.ts` to match §4 when shipping — extend mapper `toNotification()` to include new optional fields without breaking old documents (defaults).

---

## 8. Relationship to workflow

Workflow-driven alerts (job overdue, quote accepted) may **either**:

- Emit **domain** `groundzy_events` first, then a **delivery projection** with same `notifications` shape, or  
- Share the same **`type`** / **`actions`** patterns as intelligence for a **unified inbox**.

Workflow is **not** required to use `intelligence.*` event types — but **should** use the same **notification data model** for UI consistency.

---

## 9. Related documents

| Doc | Topic |
|-----|--------|
| [`notification-channel-routing.md`](./notification-channel-routing.md) | Severity → channel decision engine |
| [`notification-system.md`](./notification-system.md) | As-built constraints |
| [`../09-business/intelligence-tier-matrix.md`](../09-business/intelligence-tier-matrix.md) | Tier vs SMS/email |
