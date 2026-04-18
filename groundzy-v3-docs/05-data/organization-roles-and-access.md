# Organization roles & access (v3 system definition)

**Status:** Normative (v3)  
**Purpose:** Define how **organization (team) roles** integrate into the Groundzy v3 **access model**—policy-first, server-authoritative, event-oriented—so engineering can implement **without** inferring semantics from legacy UI helpers or duplicated Firestore fields.  
**Audience:** Backend, client, rules, and security reviewers.

**Related:** **[Unified permission model (v3) — locked decisions](./unified-permission-model-v3.md)** (reads vs writes, canonical role, mental model, examples). Audited codebase summary: [`teams-and-roles-overview.md`](../../features/teams-and-roles-overview.md). **Concrete policy:** [`org-action-policy-matrix.md`](./org-action-policy-matrix.md). **Product-facing summary (roles, CRUD, nav, assignment):** [`roles-permissions-product-spec.md`](./roles-permissions-product-spec.md). Also: [`permissions.md`](./permissions.md), [`relationships.md`](./relationships.md), [`../../architecture/Groundzy v3 — Access & Permission System.md`](../../architecture/Groundzy%20v3%20—%20Access%20%26%20Permission%20System.md).

---

## 1. Role model in v3 (definition)

### What a team role is

In v3, an **organization role** (`TeamRole`) is:

1. **An input to policy** — It is one of the attributes on **`AccessActor`** used when evaluating `can(actor, action, resource)` for org-scoped resources (CRM entities, org trees, workflow records owned by the org, team administration, billing).

2. **A named profile over capabilities** — It is **not** a free-form label. Each role maps to a **small, versioned capability profile** (see §3) that answers “what org-scoped *actions* are allowed *by default* for this membership,” before resource-specific overrides (sharing, assignment, participation).

3. **Not** a substitute for resource relationship — A user can be **`viewer`** in the org and still be a **participant** on a workflow record; those facts compose in policy (§7). The role does not replace **`tree_permissions`**, **assignee**, or **participant** relationships.

### What a team role is not

- **Not** the only field that grants access to a record. Record access always requires **relationship** (org scope, tree scope, share, participant, etc.) as defined in the v3 access doc.
- **Not** a second permission system parallel to `can()`. Any “matrix” exists only as an **implementation of the policy engine’s org-role profile**, not as a separate product concept.
- **Not** an entitlement. **Subscription tier** controls **which product surfaces exist globally** (drawers, pipelines); **role** controls **what actions are allowed within the org relationship** once the user is already a member.

**Summary:** Roles are **policy inputs** that select a **declarative capability profile** for org-scoped actions. They are **stable product concepts** (`owner` … `viewer`), not arbitrary tags.

---

## 2. Source of truth

### Canonical membership and role

| Concept | Canonical storage | Semantics |
|--------|-------------------|-----------|
| **Membership in a team** | `teams/{teamId}.members` (UID list) + user’s **single** org pointer | User belongs to **at most one** business org in the current product model: `users/{uid}.organizationId === teamId`. |
| **Role for that membership** | **`teams/{teamId}.memberRoles[uid]`** | The **only authoritative** value for “what `TeamRole` does this user have in this team.” |

### Derived field: `users.role`

**`users/{uid}.role` is a derived, denormalized cache** of the canonical role **for the user’s current `organizationId`**.

- **MUST** equal `teams/{users.organizationId}.memberRoles[uid]` whenever `organizationId` is set to a valid team.
- **MUST** be updated in the **same transactional write** as `teams.memberRoles` for that user (same pattern as membership mutations today).
- **Purpose of the cache:** Efficient Firestore rules (`hasRole`, `isOwnerOrAdmin`) and client display without loading the full team document on every read. It is **not** a second independent opinion.

### Invariant (engineering contract)

```
IF user.organizationId == teamId AND team.members contains uid
THEN user.role MUST == team.memberRoles[uid]
```

Violations are **data bugs**; repair by syncing from **`teams.memberRoles`** (canonical). New code **SHOULD** build **`AccessActor`** for policy using canonical team data when the server already loads the team document; the user cache remains valid **only if** the invariant holds.

### Solo / non-team users

Users without a business team may have **no** `organizationId` or use **solo / personal org** patterns described in the v3 access doc. **`users.role` in that context is not a `TeamRole` in the team sense**; policy **MUST** branch on **`org membership present`** vs **solo** before interpreting `TeamRole` profiles. (Exact solo patterns: see access doc Pro-solo workflow; **do not** treat unknown role as full org `owner` in team contexts.)

**Explicit gap (until specified):** If legacy data uses `users.role` without a matching `teams` doc, treat as **forbidden** for org-role policy until repaired—**do not** guess.

---

## 3. Role → permission mapping strategy

### No authoritative UI matrices

Client modules that expose large boolean grids (e.g. per-entity CRUD maps) **MUST NOT** be the **authority** for security. They may exist **only** as:

- Thin wrappers that call the **same** `can()` / server access types as the server, or
- **Optimistic** affordances that are **replaced** by server responses.

The **authority** is:

```ts
function can(actor: AccessActor, action: Action, resource?: ResourceContext): boolean
```

### How roles feed `can()`

1. **Build `AccessActor`** (server): Load user, entitlements (tier), **`organizationId`**, and **canonical org role** from **`teams/{orgId}.memberRoles[uid]`** (fallback: repair from cache if invariant enforced).
2. **Classify action** as:
   - **Org administration** (invites, role changes, team settings, billing ownership) — gated primarily by **`TeamRole` profile**.
   - **Org-scoped CRM / tree / workflow** — gated by **resource relationship** (org owns record, tree in org, etc.) **and** **`TeamRole` profile** **and** where applicable **participant / assignee / tree_permission** rules.
   - **Global surface** (e.g. opening CRM as a nav entry) — gated by **entitlements** from tier **after** auth; **role does not substitute for tier.**

3. **Map role to a capability profile** (single registry in code, versioned):

   - **`orgRoleCapabilities: Record<TeamRole, Set<OrgAction>>`** or equivalent **declarative** structure maintained in one module.
   - **Examples of `OrgAction`** (illustrative; final enum lives in code):

     | Example `OrgAction` | `viewer` | `member` | `manager` | `admin` | `owner` |
     |---------------------|----------|----------|-----------|---------|---------|
     | `org.tree.read` | yes | yes | yes | yes | yes |
     | `org.tree.update` | no | yes | yes | yes | yes |
     | `org.tree.delete` | no | no | yes | yes | yes |
     | `org.workflow.request.create` | no | yes | yes | yes | yes |
     | `org.team.invite` | no | no | no | yes | yes |
     | `org.billing.manage` | no | no | no | no | yes |

   - **Concrete v3 examples:**
     - **“Manager can delete tree”** — `can(actor, "org.tree.delete", { treeId })` is true iff the actor has an **org relationship** to the tree (e.g. `tree.organizationId === actor.orgId`), **and** `org.tree.delete` is in the profile for **`actor.orgRole === manager`**, **and** Firestore rules / server mutation path allows the delete.
     - **“Viewer can read but not mutate”** — `can(actor, "org.tree.read", …)` true; `can(actor, "org.tree.update", …)` false for viewer; participant/share paths may still allow **non-org** reads per §7.

**Precedence:** For a given resource, **resource-specific relationship** (e.g. `tree_permissions`) may **narrow or widen** behavior per the unified access doc; **org role** never overrides **explicit deny** from server policy when the server response is authoritative.

---

## 4. Access execution order (v3)

This section **normatively** aligns org roles with the v3 stack. **Tier is not access**; **rules are not the whole policy story**—but rules still **bound** what any client can do directly in Firestore.

### Layered resolution (normative)

| Order | Layer | What happens | Where org role applies |
|------|--------|----------------|-------------------------|
| **1** | **Authentication** | Reject if not signed in (or invalid server session). | N/A |
| **2** | **Hard boundaries (Firestore reads + server mutations)** | **Reads:** Security Rules enforce **coarse** membership/participant/collaborator boundaries. **Writes:** **Server-authoritative** composed policy for org + tree + workflow (`orgRoleAllows`, `canOrgOrTreeAction`, workflow gates); Rules are **not** the full matrix — see [**unified-permission-model-v3.md** §1](./unified-permission-model-v3.md). Derived **`users.role`** used in rules for Phase D helpers; canonical role remains **`teams.memberRoles`**. |
| **3** | **Resource relationship** | Determine how the actor relates to the **specific** record (org owner, participant, assignee, `tree_permissions` collaborator, public read, etc.). | Establishes **whether** org role is relevant (org-scoped resource vs share-only). |
| **4** | **Policy** `can(actor, action, resource)` | Combine **AccessActor** (includes **canonical org role**), relationship, and action. | **Primary** place where **viewer / manager / member** distinctions are expressed for org-scoped behavior. |
| **5** | **Entitlements (effective subscription tier)** | Gate **global** product capabilities (which modules exist, pipeline availability by tier rules). | **Cannot** grant org admin powers; **can** hide entire surfaces (e.g. CRM) if tier disallows. |
| **6** | **UI** | Render from **server access payload** + entitlements; **never** invent new gates. | Buttons/drawers **reflect** policy output; optional **optimistic** state only. |

### How roles participate

- **Layer 2 (rules):** Org **membership** + **owner/admin** elevation for **team document** and similar—**not** full role granularity.
- **Layer 4 (policy):** **Full `TeamRole` profile** for org-scoped actions.
- **Layer 5 (tier):** Unaffected by role; **Pro vs Teams** etc. controls **whether** the actor **has** CRM/workflow features at all.

**Consistency rule:** If the UI shows an action as allowed, **either** the server access object says so **or** it is explicitly **optimistic** pending server confirmation. **Never** the reverse for record-scoped destructive actions.

---

## 5. Viewer role decision

### Decision: **Fully support `viewer` in `AccessActor`**

**Rationale:**

- **`viewer`** is a first-class **`TeamRole`** in product types and flows (e.g. default invite posture via team **default permissions**).
- Policy and UX must distinguish **read-only org members** from **members** who can mutate org workflow/CRM entities.

### Normative typing change

Extend the v3 **`AccessActor.orgMemberships`** role union to include **`viewer`**:

```ts
role: "owner" | "admin" | "manager" | "member" | "viewer";
```

**Remove ambiguity:** Any document (including the canonical access architecture doc) that omits **`viewer`** is **incomplete** relative to this definition. **Do not** map production `viewer` users onto `member` in the actor model.

---

## 6. Server authority

### MUST be enforced server-side

- **Membership and role changes** (invite accept, role update, remove, leave, ownership transfer).
- **Billing and subscription actions** tied to the team owner.
- **Authoritative access for record views** in the v3 pattern: workflow **GET** routes return **typed `access`** (read/update/comment/…) computed with **Admin SDK** + policy.
- **Mutations** that persist org-scoped business data where **viewer** must not write—**either** deny in rules **or** in server **before** write (rules preferred where feasible for direct client writes).
- **Invitation and join** flows that set **initial role**—server is source of truth for seat limits and role assignment.

### MAY remain UI-only (with constraints)

- **Presentation:** Hiding menus, badges, empty states—driven by **server `access`** and entitlements.
- **Optimistic controls:** Temporary button state **until** server responds; **MUST** be reconciled with server **deny**.

### Remove “UI-only permission logic” ambiguity

Legacy pattern: large client **role → boolean matrix** without server check. **v3 rule:**

| Pattern | Allowed? |
|---------|----------|
| Client **`can()`** mirroring policy for UX | Yes, **if** labeled optimistic and **not** sole authority for destructive actions |
| Client matrix **only**, no server check | **No** (for mutations and authoritative read of sensitive records) |
| Firestore read “I saw the doc” **=** access | **No** for workflow record meaning (per unified access model) |

**Normative:** For **record drawers** (`view-request`, `view-quote`, …), **server `access`** fields are **authoritative**. Org role feeds **policy** that **fills** those fields.

---

## 7. Relationship to the v3 unified access model

### `AccessActor`

- **Carries** `effectiveTier`, **entitlements**, **`orgMemberships`** (with **full** `TeamRole` including **`viewer`**), **principalKeys**.
- **Org role** contributes to **global** org-scoped capability; it does **not** automatically grant **participant** powers on someone else’s private workflow line unless relationship rules say so.

### Workflow participant model

- **Orthogonal axes:** **`TeamRole`** (org membership profile) vs **participant / assignee / shared_reader** (record relationship).
- **Composition:** `can()` evaluates **relationship roles first** where the v3 matrix applies (see access doc §8 workflow tables), then applies **org role** where the table says “org admin/member” — that phrase in the **workflow policy matrix MUST** be interpreted per this doc as “org member **whose** `TeamRole` profile includes the required workflow actions.” **Viewer** typically **does not** map to “org admin/member” rows that imply edit/comment on org workflow unless explicitly added to the profile.

**Explicit follow-up for engineering:** Reconcile the **workflow policy matrix** rows labeled “Org admin/member” with **`TeamRole` profiles** so **viewer** is excluded from mutate rows by default—**without** inventing new role names.

### `tree_permissions` / sharing

- **Tree access** can come from **org membership** (same `organizationId` as tree), **`tree_permissions`**, or **public** rules.
- **Precedence** for trees follows the visibility / permission architecture (owner scope → **team role** → **explicit tree permission** → public summary). **Org role** (`viewer`) applies **before** falling through to collaborator roles when the user is an org member.

**Normative:** Sharing never **replaces** org role; they **compose**. Example: **viewer** in org + **editor** on `tree_permissions` → policy **MUST** define whether editor overrides read-only org default for that tree (recommended: **tree_permission** can grant **additional** scoped write when explicitly granted).

---

## 8. Migration strategy

**Goal:** Move from **dual storage** (`users.role` + `teams.memberRoles`) **without** breaking invites, existing users, or teams.

### Phase A — Document and assert (non-breaking)

- Declare **`teams.memberRoles`** canonical (this doc).
- Add **server-side invariant checks** on membership writes: after any change, **`users.role`** matches canonical.
- Add **optional** telemetry/logging when a read path sees mismatch (repair job candidate).

### Phase B — Policy and API read path (non-breaking)

- Implement **`AccessActor`** construction for workflow/CRM routes by loading **canonical** role from **team** when **`organizationId`** present.
- Return **typed `access`** that **does not** rely on client `getPermissionsForRole` alone.

### Phase C — Client convergence (non-breaking)

- Refactor UI to use **server `access`** and shared **policy module** (same functions as server where possible).
- Deprecate direct use of legacy matrices for **authorization** decisions; keep for **layout** only if unavoidable during transition.

### Phase D — Firestore rules (incremental, coordinate with PM/security)

- Tighten rules for **specific collections** where **viewer** must not write—**only** after server paths are verified and metrics show no legitimate client dependency on old behavior.
- **Do not** remove **dual write** of `users.role` until rules no longer depend on it **or** rules are updated to read team doc via **get()** where performance allows (tradeoff: rule complexity / cost).
- **Workflow collections** (`requests`, `quotes`, `jobs`, `invoices`): non-participant reads on **team-owned** docs require **`member`+** via **`hasRole`** (see `workflowTeamOrgWorkflowRead` in `firebase/firestore.rules`), matching **`teamOrgWorkflowReadDenied`** / **`evaluateWorkflowDocumentRead`** in the app. **Unchanged:** participant principal reads, solo org (`organizationId === uid`), **`databaseCode`** alignment, global admin.

### Compatibility guarantees

- **Invites:** Continue to resolve default role via **team settings** + optional explicit **`defaultRole`**; **viewer** remains valid.
- **Existing teams:** No mass role migration required; **invariant repair** only for inconsistent rows.
- **Breaking changes prohibited** without a **flagged** release and backfill plan.

---

## 9. Open points (explicit)

Items that **require** separate product/security decisions **before** tightening rules:

1. **`OrgAction` enum** — defined in [`org-action-policy-matrix.md`](./org-action-policy-matrix.md); implement **`orgRoleAllows`** to match §2 there.
2. **Whether `tree_permission` editor** can override **org viewer** for mutations on a specific tree (recommended yes for **explicit** grants; must be encoded in policy).
3. **Solo / Home** users: full matrix for when `users.role` is absent or non-team—**must** stay consistent with Pro-solo workflow in the access doc.

---

## Related links

- [Teams & roles — implementation audit](../../features/teams-and-roles-overview.md)
- [OrgAction policy matrix](./org-action-policy-matrix.md)
- [Permissions & access (execution order)](./permissions.md)
- [Groundzy v3 — Access & Permission System (canonical policy & server contract)](../../architecture/Groundzy%20v3%20—%20Access%20%26%20Permission%20System.md)
- Data model overview: [`data-model-overview.md`](./data-model-overview.md)
