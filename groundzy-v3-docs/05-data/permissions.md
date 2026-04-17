# Permissions & access

**Authoritative** access control for direct Firestore reads/writes: **`firebase/firestore.rules`**. **Subscription tier** (Home, Plus, Pro, Teams) is enforced primarily in **app code** (`lib/utils/tier-utils.ts`, `lib/drawers.ts`) and **user/team subscription fields**, not as a single `tier` field in every rule match.

**Organization roles (v3):** [System definition — roles in the access model](./organization-roles-and-access.md) · [OrgAction policy matrix](./org-action-policy-matrix.md) · [Teams & roles — implementation audit](../features/teams-and-roles-overview.md).

---

## Execution order (authority)

When layers conflict, **this is the resolution order** (same story everywhere in v3 docs—do not invent a second ordering per feature):

| Order | Layer | Role |
|-------|--------|------|
| 1 | **Firestore security rules** | **Hard security.** Deny unsafe reads/writes regardless of what the UI shows. Client cannot override. |
| 2 | **Org / team membership & roles** | Who may act **inside** an organization (`organizationId`, `user.role`, `teams.memberRoles`, etc.). |
| 3 | **Subscription tier (effective)** | **Product entitlements:** limits, which **features/drawers** exist (`getEffectiveSubscriptionTier`, `visibleForTiers`). **Not** a substitute for (1). |
| 4 | **UI** | Visibility, empty states, upsell—**never** described as the security boundary. |

**Implication:** A user may **see** a tier-gated drawer only if rules **also** allow the underlying data access; tier alone does not secure data.

---

## 1. User roles (Firestore)

From **`firestore.rules`** helpers:

| Concept | Mechanism |
|---------|-----------|
| **Authenticated** | `request.auth.uid` present |
| **Global admin** | Document `admins/{uid}` exists |
| **Organization / team** | `users/{uid}.organizationId` equals **team document id** (`isInTeam(teamId)`) |
| **Role on team** | `users/{uid}.role` in `['owner','admin',...]` — used with `hasRole` / `isOwnerOrAdmin` |
| **Team membership** | `teams/{teamId}.members` array + `memberRoles` map (`types/team.ts`) |

**Inconsistency note:** Signup maps **staff** → `member` (`signupRoleToCanonical` in `types/team.ts`); rules use `user.role` — ensure user doc **`role`** is set consistently for team users.

---

## 2. Team roles (product)

`TeamRole`: **owner**, **admin**, **manager**, **member**, **viewer** (`types/team.ts`).  
Rules use **owner/admin** for elevated team operations; finer roles may be partially enforced in app UI.

---

## 3. Tree access

| Path | Rule idea (simplified) |
|------|-------------------------|
| **Org member** | User’s `organizationId` matches tree’s `organizationId` |
| **Solo database** | `tree.databaseCode` matches user’s `databaseCode` |
| **Home / personal org id** | `tree.organizationId == request.auth.uid` pattern in helpers |
| **Shared collaborator** | `tree_permissions/{treeId}/members/{userId}` with `status == 'active'` |
| **work_items** | `workItemReferencedTreeAccess` / `workItemSharedAccess` tie to tree access |

**Deprecated:** `shared_data` — rules comment points to **tree_permissions**.

---

## 4. Tier-based access (product, not every rule)

| Layer | Behavior |
|-------|----------|
| **Client** | `getEffectiveSubscriptionTier(userDoc)` — Pro/Teams require **active/trialing** Stripe status |
| **Drawers** | `visibleForTiers` in drawer metadata |
| **Firestore** | Often **org membership** or **tree ownership**, not “Pro” string—**security must not rely on tier alone** on client |

**Inconsistency:** Tier gating in **UI** can diverge from **rules** if a client bypasses drawers; rules must **deny** unsafe writes regardless of tier display.

---

## 5. Data visibility (sensitivity)

- Tree history may include **cost fields** with masking flags (`ServiceRecord.dataSensitivity` in `types/tree.ts`).
- **Public** trees use **`tree_public`**, **`tree_public_summaries`**, reduced fields — separate from full `trees` read.

---

## 6. Inconsistencies to resolve in v3

| Issue | Detail |
|-------|--------|
| **Tier vs org rules** | Tier is **commercial**; Firestore rules use **membership** and **tree permissions**—document the matrix for engineers. |
| **Role field** | `user.role` vs `team.memberRoles` — two sources for role semantics. |
| **work_items** | Complex `workItemSharedAccess` — caps tree list length in helper (see rules) — potential edge cases for many `treeIds`. |

---

## Related

- [`relationships.md`](./relationships.md)
- [`data-model-overview.md`](./data-model-overview.md)
- [`organization-roles-and-access.md`](./organization-roles-and-access.md), [`org-action-policy-matrix.md`](./org-action-policy-matrix.md), [`teams-and-roles-overview.md`](../features/teams-and-roles-overview.md)
- `lib/utils/tier-utils.ts`, `firebase/firestore.rules`
