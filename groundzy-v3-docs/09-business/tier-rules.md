# Tier rules

How **access** is decided: **runtime tier names**, **subscription state**, **UI gates**, and **Firestore rules** (different layers).

---

## 1. Two naming systems (inconsistency)

| System | Values |
|--------|--------|
| **Signup / `PlanTier`** | `home` \| `pro` \| `teams` only — **no Plus** |
| **Runtime `SubscriptionTier`** (`lib/drawer-registry.ts`) | `Home` \| `Plus` \| `Pro` \| `Small Team` \| `Mid Team` \| `Large Team` \| `Enterprise` |

**Product + engineering** must map the same customer across these names; v3 should **reconcile** (`Groundzy v3/01-product/tier-system.md`).

---

## 2. Effective access (paid tiers)

From **`lib/utils/tier-utils.ts`:**

- **`getUserSubscriptionTier(userDoc)`** — reads `userDoc.subscription.tier` (raw).
- **`getEffectiveSubscriptionTier(userDoc)`** — for **Home**, unchanged; for **Plus / Pro / team tiers**, returns the tier **only if** `subscription.status` is **`active`** or **`trialing`**; otherwise effectively **Home** for gating.

**Implication:** User can have `subscription.tier: "Pro"` in Firestore but **see Home features** until payment succeeds or status updates.

**Helpers:** `hasActivePaidSubscription`, `hasPaidTierIncomplete` — redirect / upsell flows.

---

## 3. Drawer and feature gates (client)

- **`visibleForTiers`** on drawer metadata (`lib/drawers.ts`, `lib/drawer-registry.ts`) — controls which **drawers** appear.
- **Hooks:** `useProOrTeamsAccess`, `useTeamsOnlyAccess`, `useIsPlusOnlyProperties`, etc. — feature-specific.

**Firestore rules** generally use **organization membership**, **databaseCode**, **tree permissions**—**not** subscription tier strings. **Security** must not rely on client-only tier checks.

**How tier relates to rules (order of authority):** [`../05-data/permissions.md`](../05-data/permissions.md) § *Execution order (authority)*.

---

## 4. Team subscription

- **Team** documents carry subscription fields (`types/team.ts`); team-scoped billing may parallel user subscription—**exact** split is product-specific; engineers should trace **Stripe webhooks** for whether team vs user doc is updated.

---

## 5. Special cases (repo)

| Case | Behavior |
|------|----------|
| **Plus** | Yearly Stripe path (`lib/stripe-config.ts`); **does not** unlock Pro **clients/properties** drawers—those are **`Pro` + team tiers** only (`lib/drawers.ts`, `clients-properties`). Plus sits between Home and Pro for **limits and upgrade path**, not CRM. |
| **Hire a Pro** | Shown for **Home/Plus**, hidden for **Pro/Teams** (`docs/features/hire-pro.md`) — revenue vs positioning choice |
| **Enterprise** | Placeholder pricing null in struct; may map to Large Team price in `getTeamTierName` for Stripe (`stripe-config.ts` comment) |

---

## 6. Alignment checklist (v3)

- [ ] One **canonical** tier enum for product + analytics + Firestore fields (or explicit mapping table).
- [ ] **Effective tier** applied everywhere user-facing limits are enforced.
- [ ] **Rules** documented next to **tier** docs so CS/engineers know what is enforceable where.

---

## Related

- [`pricing.md`](./pricing.md)
- [`growth-strategy.md`](./growth-strategy.md)
- `firebase/firestore.rules`
