# Permissions & access

**v3 unified model (locked):** [Unified permission model (v3)](./unified-permission-model-v3.md) — **reads** = coarse Firestore rules + optional server loaders; **writes** = server-authoritative policy (`orgRoleAllows`, `canOrgOrTreeAction`, workflow gates). Rules are **not** a full duplicate of composed org + tree + workflow logic.

**Direct Firestore:** **`firebase/firestore.rules`** bound what the client can read/write without Admin SDK. **Subscription tier** (Home, Plus, Pro, Teams) is enforced primarily in **app code** (`lib/utils/tier-utils.ts`, `lib/drawers.ts`) and **user/team subscription fields**, not as a single `tier` field in every rule match.

**Organization roles (v3):** [System definition — roles in the access model](./organization-roles-and-access.md) · [OrgAction policy matrix](./org-action-policy-matrix.md) · [Teams & roles — implementation audit](../features/teams-and-roles-overview.md).

---

## Execution order (authority)

When layers conflict, **this is the resolution order** (same story everywhere in v3 docs—do not invent a second ordering per feature):

| Order | Layer | Role |
|-------|--------|------|
| 1 | **Firestore security rules** | **Coarse hard boundary** on **reads** (membership, participant index, collaborator status, solo/dbCode paths). On **writes**, rules are a **safety net** while **composed** policy (org + tree + participant) lives on the **server** for authoritative mutations — see [unified-permission-model-v3.md](./unified-permission-model-v3.md) §1. |
| 2 | **Org / team membership & roles** | Who may act **inside** an organization. **Canonical role:** `teams.memberRoles[uid]`; **`users.role`** is cache — see [organization-roles-and-access.md](./organization-roles-and-access.md) §2. |
| 3 | **Subscription tier (effective)** | **Product entitlements:** limits, which **features/drawers** exist (`getEffectiveSubscriptionTier`, `visibleForTiers`). **Not** a substitute for (1). |
| 4 | **UI** | Visibility, empty states, upsell—**never** described as the security boundary. |

**Implication:** A user may **see** a tier-gated drawer only if rules **also** allow the underlying data access; tier alone does not secure data. **Mutations** that depend on **full** org-role composition **must** go through server-validated paths (API / Admin SDK); do not rely on Security Rules alone for those matrices.

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
Rules use **owner/admin** for elevated **team document** operations; **viewer vs member+** on org-scoped **CRM / trees** writes is enforced in **`firestore.rules`** (Phase D helpers) and overlapping server tree routes — see **`permissions-utils` / `org-action.ts`** for the product matrix.

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

## 6. Inconsistencies — resolved posture (v3)

| Issue | Resolution |
|-------|------------|
| **Reads vs writes / rule drift** | **Locked:** [unified-permission-model-v3.md](./unified-permission-model-v3.md) §1 — **writes** are **server-authoritative** for composed policy; **reads** use **coarse** rules (+ server access envelopes where implemented). Do not attempt full matrix parity in Security Rules for composition-heavy paths. |
| **Tier vs org rules** | CRM/trees Phase D uses **`teamOrgMutateMemberPlus`** / **`teamOrgMutateManagerPlus`** in **`firebase/firestore.rules`**. Workflow collections use **`workflowTeamAccess`** / participant reads; **narrow client writes** over time per unified model; staging metrics remain useful before tightening legacy client paths (`teams-and-roles-overview.md` §5). |
| **`users.role` vs `teams.memberRoles`** | **`teams.memberRoles`** is canonical; **`users.role`** is cache — transactional updates required; drift = bug; **`logTeamMemberRoleDriftIfAny`** (`lib/groundzy/policy/team-role-invariant.ts`); optional background repair — [organization-roles-and-access.md](./organization-roles-and-access.md) §2, [unified-permission-model-v3.md](./unified-permission-model-v3.md) §2. |
| **work_items / recurring_plans thresholds** | **Classified:** **system-class** indices vs **CRM-class** entities — delete bars differ **by design** ([unified-permission-model-v3.md](./unified-permission-model-v3.md) §4). **`work_items`** shared-tree list caps in rules remain a separate edge-case follow-up. |
| **Collaborator vs org UX** | **Guidelines:** [collaborator-org-role-ux.md](../features/collaborator-org-role-ux.md) — always **Org:** vs **On this tree:** labels. |

**Implementation inventory:** [role-capabilities-matrix.md](../features/role-capabilities-matrix.md) maps capabilities to this posture (reads vs server writes).

---

## Related

- [`relationships.md`](./relationships.md)
- [`data-model-overview.md`](./data-model-overview.md)
- [`organization-roles-and-access.md`](./organization-roles-and-access.md), [`org-action-policy-matrix.md`](./org-action-policy-matrix.md), [`teams-and-roles-overview.md`](../features/teams-and-roles-overview.md)
- `lib/utils/tier-utils.ts`, `firebase/firestore.rules`
