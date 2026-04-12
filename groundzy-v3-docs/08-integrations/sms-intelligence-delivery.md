# SMS — intelligence & alert delivery

**Status:** Specification — **not** implemented until product, legal, and carrier constraints are satisfied.  
**Depends on:** [`../07-systems/alert-notification-system.md`](../07-systems/alert-notification-system.md), [`../07-systems/delivery-preferences-and-routing.md`](../07-systems/delivery-preferences-and-routing.md), [`../09-business/intelligence-tier-matrix.md`](../09-business/intelligence-tier-matrix.md).

---

## 1. Role

SMS is for **high-urgency, short** messages — typically **Teams / premium** tiers. It is **not** a full duplicate of in-app intelligence; it is a **channel** for a subset of severities and user opt-ins.

---

## 2. Constraints

| Area | Requirement |
|------|----------------|
| **Consent** | Explicit opt-in; STOP/HELP per regional rules |
| **Tier** | Capability flag (e.g. `canUseSmsAlerts`) — see tier matrix |
| **Rate limits** | Per user and per org to control cost and spam |
| **Content** | Short template; link to app for detail — no long evidence in SMS body |
| **PII** | Minimize tree address in SMS if policy requires |

---

## 3. Integration shape

- **Producer:** same `intelligence.alert_triggered` → routing layer decides SMS vs not.
- **Provider:** TBD (Twilio, etc.) — document in this file when chosen.
- **Delivery record:** optional `notification.created` with `channel: "sms"` or provider message id stored for support.

---

## 4. Relationship to email

[`email.md`](./email.md) covers transactional Resend today. **Digest and intelligence email** may share preference infrastructure with SMS routing (same user preference doc, different channel flags).

---

## 5. Open decisions

- Provider and webhook callbacks for delivery/failure
- Whether SMS sends require **separate** Firestore rows from in-app `notifications`
- Retry policy and dead-letter handling
