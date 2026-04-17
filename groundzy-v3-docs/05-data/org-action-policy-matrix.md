# OrgAction policy matrix (v3 — concrete `can()` contract)

**Status:** Normative (v3)  
**Purpose:** Define the **complete, closed** `OrgAction` vocabulary and **authoritative** `TeamRole → OrgAction` mapping so engineers implement **one** policy module without inventing string literals or duplicating matrices.  
**Depends on:** [`organization-roles-and-access.md`](./organization-roles-and-access.md), [`permissions.md`](./permissions.md), [**unified-permission-model-v3.md**](./unified-permission-model-v3.md) (locked reads/writes posture, evaluation examples).

**← Audited state:** [`teams-and-roles-overview.md`](../../features/teams-and-roles-overview.md) (what the codebase does today).

**Legacy alignment:** CRM + workflow CRUD gates for team members match the current **`lib/permissions-utils.ts`** behavior; team administration flags match **`inviteMembers`**, **`changeRoles`**, **`removeMembers`**, **`teamSettings`**, **`billing`**.

---

## 1. OrgAction enum (authoritative)

### 1.1 Naming rules

- **Prefix:** Every constant is a **single string** starting with `org.`.
- **Separation:** `OrgAction` governs **organization-membership capability** (what a `TeamRole` allows **inside the org** on org-scoped data). It does **not** replace **workflow row** semantics (participant / assignee / shared reader) — see §3–4.
- **Exhaustiveness:** Implementations MUST define this as a closed **TypeScript `const` enum or union** derived from this list — **no new `org.*` literals** without a doc PR that updates this section.

### 1.2 Complete list

#### CRM (org-owned property / client / tree records)

| OrgAction |
|-----------|
| `org.tree.read` |
| `org.tree.create` |
| `org.tree.update` |
| `org.tree.delete` |
| `org.property.read` |
| `org.property.create` |
| `org.property.update` |
| `org.property.delete` |
| `org.client.read` |
| `org.client.create` |
| `org.client.update` |
| `org.client.delete` |

#### Workflow entities (org-scoped pipeline records)

Same CRUD vocabulary for each entity:

| OrgAction |
|-----------|
| `org.workflow.request.create` |
| `org.workflow.request.read` |
| `org.workflow.request.update` |
| `org.workflow.request.delete` |
| `org.workflow.quote.create` |
| `org.workflow.quote.read` |
| `org.workflow.quote.update` |
| `org.workflow.quote.delete` |
| `org.workflow.job.create` |
| `org.workflow.job.read` |
| `org.workflow.job.update` |
| `org.workflow.job.delete` |
| `org.workflow.invoice.create` |
| `org.workflow.invoice.read` |
| `org.workflow.invoice.update` |
| `org.workflow.invoice.delete` |

#### Team administration & billing

| OrgAction |
|-----------|
| `org.team.member.invite` |
| `org.team.member.role_update` |
| `org.team.member.remove` |
| `org.team.settings.update` |
| `org.team.invite_code.create` |
| `org.team.invite_code.revoke` |
| `org.team.invite_code.regenerate_primary` |
| `org.team.membership.self_leave` |
| `org.team.ownership.transfer` |
| `org.billing.manage` |

**Count:** 39 `OrgAction` values (12 CRM + 16 workflow + 11 team/billing).

### 1.3 Out of scope for `OrgAction`

The following are **not** `OrgAction` strings — they belong to **WorkflowAction**, **TreeShareAction**, **PublicAccess**, or route-specific policy:

- Comment-only, upload-only, approve, pay, schedule — see **Groundzy v3 — Access & Permission System** workflow matrices.
- Accepting an **invite** (`joinTeam`) — gated by **valid invite + seats**, not role profile.
- **Global entitlements** (`workflowPipeline`, `crmClients`, …) — tier layer; **must** pass before `OrgAction` matters for UI surfaces.

---

## 2. Role → OrgAction mapping (FINAL)

**Legend:** `✓` = allowed, `✗` = denied (for actors who are **only** evaluated by org-role profile on **org-scoped** resources).

| OrgAction | viewer | member | manager | admin | owner |
|-----------|--------|--------|---------|-------|-------|
| `org.tree.read` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `org.tree.create` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.tree.update` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.tree.delete` | ✗ | ✗ | ✓ | ✓ | ✓ |
| `org.property.read` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `org.property.create` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.property.update` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.property.delete` | ✗ | ✗ | ✓ | ✓ | ✓ |
| `org.client.read` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `org.client.create` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.client.update` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.client.delete` | ✗ | ✗ | ✓ | ✓ | ✓ |
| `org.workflow.request.create` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.request.read` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.request.update` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.request.delete` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.quote.create` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.quote.read` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.quote.update` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.quote.delete` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.job.create` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.job.read` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.job.update` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.job.delete` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.invoice.create` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.invoice.read` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.invoice.update` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.workflow.invoice.delete` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `org.team.member.invite` | ✗ | ✗ | ✗ | ✓ | ✓ |
| `org.team.member.role_update` | ✗ | ✗ | ✗ | ✓ | ✓ |
| `org.team.member.remove` | ✗ | ✗ | ✗ | ✓ | ✓ |
| `org.team.settings.update` | ✗ | ✗ | ✗ | ✓ | ✓ |
| `org.team.invite_code.create` | ✗ | ✗ | ✗ | ✓ | ✓ |
| `org.team.invite_code.revoke` | ✗ | ✗ | ✗ | ✓ | ✓ |
| `org.team.invite_code.regenerate_primary` | ✗ | ✗ | ✗ | ✓ | ✓ |
| `org.team.membership.self_leave` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `org.team.ownership.transfer` | ✗ | ✗ | ✗ | ✗ | ✓ |
| `org.billing.manage` | ✗ | ✗ | ✗ | ✗ | ✓ |

**Notes (normative):**

- **`viewer`:** Read **trees / properties / clients** only; **no** workflow OrgActions; **no** team admin except **self leave**.
- **`member`–`manager`:** Workflow OrgActions **identical** for these roles in this matrix; difference is **CRM delete** (`org.tree|property|client.delete`) starting at **manager**.
- **`admin` vs `owner`:** Only **`org.billing.manage`** and **`org.team.ownership.transfer`** differ.
- **`org.team.membership.self_leave`:** **Owner** **✗** — ownership must change via **`org.team.ownership.transfer`** first.

### 2.1 Implementation requirement

Code MUST expose:

```ts
function orgRoleAllows(role: TeamRole, action: OrgAction): boolean
```

as a **pure** function matching the table above **exactly**.

**Implementation (Groundzy app):** [`lib/groundzy/policy/org-action.ts`](../../../app/lib/groundzy/policy/org-action.ts) (`ORG_ACTIONS`, `orgRoleAllows`, `workflowEntityToOrgReadAction`, `workflowEntityToOrgUpdateAction`, `teamOrgWorkflowReadDenied`, `isOrgAction`). UI shim: [`lib/permissions-utils.ts`](../../../app/lib/permissions-utils.ts).

---

## 3. Relationship overrides

Section 2 defines **`orgRoleAllows(role, action)`** only. Effective authorization **also** depends on **how** the actor relates to the **specific resource**.

### 3.1 Applicability

| Context | Does `OrgAction` apply? |
|---------|-------------------------|
| Actor is **org member** (`users.organizationId === resource.organizationId` for org-owned resources) | **Yes** — gate mutate/read CRM + workflow OrgActions with §2 **after** tier entitlements. |
| Actor accesses **only** via **`tree_permissions`** (collaborator; not org member for that business) | **No** — use **`tree_permissions` role + tree policy**, not §2. |
| Actor accesses **only** as **workflow participant / assignee / shared reader** (no org insider row) | **No** — use **workflow relationship matrix**, not §2 `OrgAction` (except where server composes both — §4). |
| **Public** tree / summary | **No** — public rules; **no** `OrgAction`. |

### 3.2 Workflow participant vs org **viewer**

| Question | Answer |
|----------|--------|
| Can **participant** “override” **org viewer** for workflow **read / comment** on a record they participate in? | **Yes.** Relationship **participant row** grants record-scoped **read / comment / upload** per v3 workflow matrix **even if** `orgRoleAllows(viewer, org.workflow.*.read)` is **false** for org-wide workflow. They still **do not** gain **org-wide** `org.workflow.*.create` from participant alone. |
| Does **org admin** “override” **participant** restrictions for **edit** on an **org-owned** record when the actor is both org insider and participant? | **Yes**, when policy classifies them as **org insider** for that record: **Org admin/member** row in the workflow matrix applies (with **viewer** excluded from that row per [`organization-roles-and-access.md`](./organization-roles-and-access.md)). |

### 3.3 `tree_permissions` vs org **viewer**

| Question | Answer |
|----------|--------|
| Can **tree editor** (or equivalent collaborator role on `tree_permissions`) override **org viewer** for mutations on **that** tree? | **Yes**, for **that tree only**: collaborator grant **OR**’s with org-role deny for **`org.tree.update`** / **`org.tree.delete`** (scoped to the tree document). |
| Does **org admin** override **collaborator** deny? | **Org insider** path is evaluated separately; Firestore rules + server policy **must** still honor **explicit collaborator scope**. **Normative:** Collaborator **cannot** expand beyond what **`tree_permissions`** grant; org role **cannot** bypass **missing** tree grant for **non-org** trees. |

### 3.4 Public access

**Public** discovery / summary surfaces **do not** consult `OrgAction`. **Authenticated** org operations on full records **do**.

---

## 4. Conflict resolution rules (CRITICAL)

Evaluate in **order**. **Stop at first hard deny** where noted.

### 4.1 Global ordering (all resources)

| Priority | Rule |
|----------|------|
| **D1** | **Authentication failed** → **deny all.** |
| **D2** | **Firestore security rules deny** (direct client write/read path) → **deny** (hard boundary). |
| **D3** | **Explicit server deny** (`forbidden`, business rule hard stop — e.g. archived entity, integrity) → **deny** regardless of matrices. |
| **D4** | **Tier entitlement missing** for the surface (e.g. CRM not entitled) → **deny** global surface (`missing_entitlement` UX); **does not** grant admin powers. |

### 4.2 Combining org role with relationships

Let **`O = orgRoleAllows(actor.teamRole, action)`** for an **`OrgAction`** when the resource is **org-scoped**.

| Priority | Rule |
|----------|------|
| **R1** | **`O === true`** **and** no higher-priority deny → **allow** org-path CRM/workflow mutation **subject to D2–D4**. |
| **`R2`** | **`O === false`** **and** actor has **only** org membership path → **deny** org-path action (viewer workflow CRUD, viewer CRM mutate). |
| **R3** | **`O === false`** **and** actor has **additional** relationship (`tree_permissions`, participant, assignee): evaluate **relationship policy**. If relationship **allows** the concrete operation → **allow** **without** requiring **`O`** for that relationship-specific capability (§3.2–3.3). |
| **R4** | **Multiple hats** (e.g. org **member** + **participant** on same workflow record): Effective workflow capabilities **union** applicable v3 workflow rows that apply to this actor; **restrictive matrix cells** still apply per **workflow doc** (participant cannot edit body where matrix says no). **Normative:** For **org-owned** records where **Org admin/member** row applies, **TeamRole ∈ {member, manager, admin, owner}** uses **Org** row; **viewer** does **not** — use **Participant** row only if they are in `participantPrincipalIds`. |

### 4.3 Precedence summary (single sentence)

**Explicit deny (D2–D4) beats everything; among allows, relationship-specific grants may satisfy an action when org-role denies; among org insiders, workflow matrix chooses the effective row — org insider row beats participant-only restriction for edit on org-owned records only when `TeamRole` qualifies as org “member-tier” per §2.**

### 4.4 Admin-vs-admin removal (business rule)

**Matrix:** **`org.team.member.remove`** is **✓** for **admin**. **Additional rule:** Admin **cannot** remove or re-role another **admin** (existing product behavior) — enforce in **server** policy as an **extra check** **after** `orgRoleAllows` is true.

---

## 5. Server enforcement mapping

**Meaning:**

- **Rules** = `firebase/firestore.rules` client direct access.
- **Server** = Cloud Functions / Next API / server actions using Admin SDK.
- **Both** = rules are necessary **and** insufficient; server must still validate business rules.

| OrgAction | Rules | Server |
|-----------|-------|--------|
| `org.tree.*` | **Both** — rules today allow broad org-member access; **tighten** over time so **viewer** cannot `update`/`delete` on `trees` (migration). Until then, **server + policy** MUST enforce **viewer** mutate deny. |
| `org.property.*`, `org.client.*` | **Both** — same pattern as trees for CRM collections. |
| `org.workflow.*.*` | **Server primary** — v3 record GET already returns **`access`**; mutations MUST run policy with **`orgRoleAllows`**. Rules may still allow reads in some paths; **authoritative** edit/delete = **server**. |
| `org.team.member.invite`, `org.team.member.role_update`, `org.team.member.remove` | **Server** — implemented via server actions today; rules block arbitrary client writes to `teams`. |
| `org.team.settings.update` | **Both** — team doc updates require owner/admin in rules + server validation. |
| `org.team.invite_code.*` | **Server** — invite codes written via Admin / controlled paths. |
| `org.team.membership.self_leave` | **Server** — `leaveTeam` server action. |
| `org.team.ownership.transfer` | **Server** — `transferOwnership`. |
| `org.billing.manage` | **Server** — Stripe / billing APIs; never client-only. |

**Summary:** Every **✓** in §2 MUST be **enforced** for **mutations** on production paths via **server policy** at minimum; **rules** SHOULD converge to match **viewer** CRM/workflow denies (see [`organization-roles-and-access.md`](./organization-roles-and-access.md) migration Phase D).

---

## 6. Migration notes

| Legacy | Replacement |
|--------|-------------|
| **`getPermissionsForRole` / `RolePermissions` in `lib/permissions-utils.ts`** | **`orgRoleAllows`** for all **`org.*`** actions + **relationship** modules for participant/tree/public. Keep a **thin shim** during migration that delegates to **`orgRoleAllows`** so no duplicate truth. |
| **Ad-hoc `userDoc.role === 'admin'` checks** | Prefer **`orgRoleAllows(role, specific OrgAction)`** + relationship checks; reserve raw role compares for **exceptions** (admin-vs-admin) **next to** policy. |
| **UI-only gating** | Replace with **server `access`** payload for record drawers; UI may mirror **`orgRoleAllows`** **only** as optimistic affordance. |
| **Firestore “member can write” without viewer split** | **Phase D:** narrow rules per collection for **viewer** vs **member** once server metrics are clean. |

**Compliance check:** Adding a new org-gated button **without** an `OrgAction` mapping is a **process failure** — add the action here first.

---

## Related

- [`teams-and-roles-overview.md`](../../features/teams-and-roles-overview.md) — audit / current implementation
- [`organization-roles-and-access.md`](./organization-roles-and-access.md)
- [`permissions.md`](./permissions.md)
- [`../../architecture/Groundzy v3 — Access & Permission System.md`](../../architecture/Groundzy%20v3%20—%20Access%20%26%20Permission%20System.md)
