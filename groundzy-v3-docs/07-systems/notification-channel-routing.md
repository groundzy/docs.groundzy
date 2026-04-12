# Notification channel routing (decision engine)

**Status:** Specification — **logical layer** between `intelligence.*` / domain events and **concrete deliveries** (in-app rows, Resend, SMS).  
**Depends on:** [`notification-data-model-v3.md`](./notification-data-model-v3.md), [`../05-data/derived-events-payload-schema.md`](../05-data/derived-events-payload-schema.md), [`../09-business/intelligence-tier-matrix.md`](../09-business/intelligence-tier-matrix.md), [`delivery-preferences-and-routing.md`](./delivery-preferences-and-routing.md).

---

## 1. Framing

You do **not** build “notifications first.”

```
Intelligence or domain event (append-only)
        ↓
Notification Decision Layer  ← this document
        ↓
Channel writer(s): app / email / SMS
```

**Inputs to the decision layer:**

1. **Event** — e.g. `intelligence.alert_triggered` with `outputs.severity`, `outputs.suggestedActions`, `dedupeKey`, `subject` (tree, job, …).
2. **Recipient resolution** — who receives this (owner, team, assignee) — [`delivery-preferences-and-routing.md`](./delivery-preferences-and-routing.md).
3. **Tier & capabilities** — what channels are **allowed** — [`intelligence-tier-matrix.md`](../09-business/intelligence-tier-matrix.md).
4. **User/org preferences** — opt-in, digest vs instant, quiet hours (when stored).

**Output:** For each `(recipientUserId, logicalAlert)` a set of **channel instructions**: create `notifications` row with `channel: app`, enqueue email, enqueue SMS, or skip.

---

## 2. Default severity → channel matrix (normative baseline)

Product may tune; document changes in [`../99-reference/decision-log.md`](../99-reference/decision-log.md) when they diverge.

| Severity | In-app | Email | SMS |
|----------|--------|-------|-----|
| `info` | Yes | No | No |
| `recommendation` | Yes | Optional (digest / user pref) | No |
| `warning` | Yes | Yes (instant or digest per pref) | No |
| `critical` | Yes | Yes | Yes (tier + opt-in; see §4) |

**Interpretation:**

- **In-app** is always the **default** carrier when the user should see the alert in Groundzy.
- **Email** adds **awareness outside the app** for `warning`+; `recommendation` email only if preferences or digest rules say so.
- **SMS** is **interrupt-only**: typically **`critical`** and **never** the sole channel for low-severity noise.

---

## 3. Decision algorithm (pseudocode)

For each resolved recipient `userId`:

```
function decideChannels(event, recipientContext):
  severity = event.outputs.severity
  tier = recipientContext.effectiveTier
  prefs = recipientContext.notificationPreferences  // may be defaults

  channels = { app: true }  // primary default for all severities we surface

  if severity == "info":
    return { app: true, email: false, sms: false }  // unless product enables email for info digests later

  if severity == "recommendation":
    email = prefs.emailDigestIncludesRecommendations OR prefs.emailInstantForRecommendations
    return { app: true, email: email, sms: false }

  if severity == "warning":
    email = prefs.email != "none" AND tierAllowsEmail(tier)
    return { app: true, email: email, sms: false }

  if severity == "critical":
    email = prefs.email != "none" AND tierAllowsEmail(tier)
    sms = tierAllowsSms(tier) AND prefs.smsCritical == true AND hasSmsConsent(userId)
    return { app: true, email: email, sms: sms }
```

**Quiet hours (optional):** If `now` in quiet window and `severity` is not `critical`, set `sms = false`; optionally delay email to next window.

**Dedupe:** Before writing, check `(dedupeKey, userId, channel)` — if already **sent** for this window, skip duplicate channel dispatch (in-app may still **upsert** read state per product).

---

## 4. Tier gating (delivery access)

SMS and sometimes **email digests** are **paid or capability-gated**. The decision layer must call the same capability source as the UI (`getCapabilitiesForTier` / subscription — see implementation plan in [`../../architecture/intelligence-implementation-plan.md`](../../architecture/intelligence-implementation-plan.md)).

| Capability (illustrative) | Effect |
|----------------------------|--------|
| `notifications.email` | Allow email branch |
| `notifications.sms_critical` | Allow SMS for `critical` |
| Home / base | In-app only unless product overrides |

Exact mapping: [`intelligence-tier-matrix.md`](../09-business/intelligence-tier-matrix.md).

---

## 5. Example flow

1. `weather.forecast.updated` (internal or ingestion) triggers evaluation.
2. `intelligence.alert_triggered` appended with `outputs.severity: critical`, `dedupeKey`, `subject: tree`.
3. **Decision layer** resolves recipient = tree owner `userId`.
4. **Matrix** says: app + email + SMS (if tier + consent).
5. **Writers:**
   - `notifications` doc `channel: app`, `sourceEventId` set, `actions` populated.
   - Enqueue Resend for email body (short + link).
   - Enqueue SMS provider for truncated text + link (if integrated).

---

## 6. Workflow-triggered alerts (same engine)

For **non-intelligence** events (e.g. job assigned, quote accepted):

- Either emit a **workflow**-scoped event first, or a thin `intelligence.alert_triggered` with `ruleId: workflow.quote_accepted_v1` — product choice.
- **Same decision matrix** can apply if `severity` is set consistently (`warning` vs `critical` for “job today”).

Keeps **one** inbox and one routing policy.

---

## 7. What exists vs missing (repo honesty)

| Piece | Today |
|-------|--------|
| Severity on Firestore `notifications` | **Missing** — add per [`notification-data-model-v3.md`](./notification-data-model-v3.md) |
| Channel field per row | **Missing** |
| Central decision function in code | **Missing** — this doc is the spec |
| Resend | **Present** — [`email.md`](../08-integrations/email.md) |
| SMS | **Not in app** — routing returns `sms: true` only when integration exists |
| User preferences doc | **Partial / TBD** — defaults can assume “email on for warning+” until UI ships |

---

## 8. Related

| Doc | Topic |
|-----|--------|
| [`alert-notification-system.md`](./alert-notification-system.md) | End-to-end pipeline |
| [`notification-projection-model.md`](./notification-projection-model.md) | Causality |
| [`../08-integrations/sms-intelligence-delivery.md`](../08-integrations/sms-intelligence-delivery.md) | SMS constraints |
