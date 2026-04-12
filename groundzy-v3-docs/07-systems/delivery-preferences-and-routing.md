# Delivery preferences & routing (“who gets notified?”)

**Status:** Specification — complements [`alert-notification-system.md`](./alert-notification-system.md) §6.  
**Depends on:** [`../01-product/tier-system.md`](../01-product/tier-system.md), [`../../architecture/Groundzy v3 - Access & Permission System.md`](../../architecture/Groundzy%20v3%20-%20Access%20%26%20Permission%20System.md) (if present).

**Channel decision matrix (severity → app/email/SMS):** [`notification-channel-routing.md`](./notification-channel-routing.md).

---

## 1. Scope

- **Preferences:** per user and optionally org defaults — channel, frequency, quiet hours.
- **Routing:** given an `intelligence.alert_triggered`, which **users** receive an in-app row, email, or SMS.

---

## 2. Channel matrix (product)

| Channel | Default | Notes |
|---------|---------|--------|
| **In-app** | On for actionable tiers | Baseline; see [`notification-projection-model.md`](./notification-projection-model.md) |
| **Email** | Opt-in / tier-gated | Digests vs instant; unsubscribe compliance |
| **SMS** | Opt-in, premium | [`../08-integrations/sms-intelligence-delivery.md`](../08-integrations/sms-intelligence-delivery.md) |

---

## 3. Frequency

| Mode | Behavior |
|------|----------|
| **Instant** | Emit on each qualifying `intelligence.*` (subject to dedupe) |
| **Daily digest** | Batch non-critical; critical may still be instant |
| **Weekly** | Lower-priority summaries |

**Quiet hours:** optional window where only `critical` (or none) pushes to mobile/SMS; email may still queue.

---

## 4. Routing dimensions

Use stable IDs from [`../05-data/entities.md`](../05-data/entities.md):

| Dimension | Examples |
|-----------|----------|
| **Ownership** | Tree owner, property owner |
| **Team / org** | Members with role ≥ threshold |
| **Workflow** | Assigned pro, job participants, quote recipients — see [`architecture/workflow-participation-external-access-v3.md`](../../architecture/workflow-participation-external-access-v3.md) |
| **Sharing** | `tree_permissions` / org share — same as record access where appropriate |

**Conflict:** multiple targets — emit **one notification per user** with shared `sourceEventId` or dedupe across users per policy.

---

## 5. Preferences storage (illustrative)

Possible locations (choose one in implementation):

- `users/{uid}.preferences.notifications` — channel toggles, frequency, quiet hours
- `teams/{tid}.settings.defaultNotifications` — org defaults with user overrides

Document final paths in [`../../reference/firestore-collections.md`](../../reference/firestore-collections.md) when implemented.

---

## 6. Audit

Store minimal **routing reason** on derived event or server log: `reason: "tree_owner" | "job_assignee"` — PII balance per product.

---

## 7. Related

- [`../09-business/intelligence-tier-matrix.md`](../09-business/intelligence-tier-matrix.md)
- [`../../features/intelligence-alerts.md`](../../features/intelligence-alerts.md)
