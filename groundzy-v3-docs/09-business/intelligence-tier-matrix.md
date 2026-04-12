# Intelligence tier matrix (evaluation vs delivery)

**Status:** Specification — product must confirm labels (Home / Plus / Pro / Teams).  
**Depends on:** [`tier-rules.md`](./tier-rules.md), [`../01-product/tier-system.md`](../01-product/tier-system.md), [`../07-systems/alert-notification-system.md`](../07-systems/alert-notification-system.md) §7.

---

## 1. Two levers

| Lever | Question |
|-------|----------|
| **Evaluation access** | Which **rules** may run for this subscription? |
| **Delivery access** | Which **channels** and **frequencies** are allowed? |

**Implementation:** capability flags from billing/subscription (e.g. `intelligenceRuleSet`, `channels.sms`, `channels.emailDigest`) — **not** scattered UI checks only.

---

## 2. Illustrative matrix (replace with product source of truth)

| Tier | Rule scope (examples) | In-app | Email | SMS | Workflow automation hints |
|------|------------------------|--------|-------|-----|---------------------------|
| **Home** | Basic weather + generic seasonal | Yes | Limited / none | No | No |
| **Plus** | + More combined rules, recommendations | Yes | Digests optional | No | Suggestions |
| **Pro** | + Tree-specific, richer combined | Yes | Yes | Optional / gated | Deeper actions |
| **Teams** | + Workflow-aware, assignment routing | Yes | Yes | Premium opt-in | Triggers, team routing |

Exact rows must match [`PRICING-REFERENCE.md`](../../PRICING-REFERENCE.md) or internal pricing docs when published.

---

## 3. Alignment with code

- **Weather card / drawer:** existing tier UX — see [`../../features/weather.md`](../../features/weather.md).
- **Entitlements:** single resolver used by evaluation runtime and API routes ([`../05-data/evaluation-runtime.md`](../05-data/evaluation-runtime.md)).

---

## 4. Related

- [`../../architecture/tier-system-audit-2026-04-08.md`](../../architecture/tier-system-audit-2026-04-08.md) — historical
- [`../07-systems/delivery-preferences-and-routing.md`](../07-systems/delivery-preferences-and-routing.md)
