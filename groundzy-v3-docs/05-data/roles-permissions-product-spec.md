# Roles & permissions — product spec (source of truth)

**Status:** Normative product reference (aligned with v3 policy)  
**Audience:** Product, design, engineering  
**Purpose:** Single document describing **what each team role is for**, **org-wide vs scoped access**, **CRUD by entity**, **navigation visibility**, **assignment/linking**, and the **intended difference between `member` and `viewer`**. Use this before changing permissions or UX.

**Authoritative policy detail:** [`org-action-policy-matrix.md`](./org-action-policy-matrix.md) (closed `OrgAction` list and exact ✓/✗ table).  
**Architecture & invariants:** [`organization-roles-and-access.md`](./organization-roles-and-access.md), [`unified-permission-model-v3.md`](./unified-permission-model-v3.md).  
**Implementation (Groundzy app):** [`lib/groundzy/policy/org-action.ts`](../../../app/lib/groundzy/policy/org-action.ts) (`orgRoleAllows`, `MATRIX`), [`lib/permissions-utils.ts`](../../../app/lib/permissions-utils.ts) (UI shim → matrix), [`hooks/useOrgWorkflowReadPermissions.ts`](../../../app/hooks/useOrgWorkflowReadPermissions.ts), [`lib/drawers.ts`](../../../app/lib/drawers.ts) (drawer metadata).

---

## 1. Roles at a glance

| Role | Org insider | CRM (trees / properties / clients) | Org-wide workflow pipeline lists | Team admin | Billing |
|------|-------------|--------------------------------------|-----------------------------------|------------|---------|
| **Owner** | Yes | Full CRUD per matrix; delete = tree/property/client with manager+ pattern | Full workflow CRUD | Yes (incl. transfer) | Yes |
| **Admin** | Yes | Same as owner except deletes follow manager+ row | Full workflow CRUD | Yes (no ownership transfer) | No |
| **Manager** | Yes | Like member **+** `delete` on trees / properties / clients | Full workflow CRUD (same as member) | No | No |
| **Member** | Yes | Read all CRM; create/update; **no** delete on tree/property/client | Full workflow CRUD | No | No |
| **Viewer** | Yes | **Read-only** CRM (no create/update/delete) | **No** org-wide workflow `OrgAction`s (see §6) | No (except self-leave) | No |

Canonical role storage: **`teams/{orgId}.memberRoles[uid]`**; **`users.role`** is a denormalized cache — see [`organization-roles-and-access.md`](./organization-roles-and-access.md) §2.

---

## 2. Org-wide vs scoped access

### 2.1 Org-wide (membership profile)

**Org-wide** means: the actor is a **team org member** on the business org (`users.organizationId` matches the resource’s org, or equivalent team scope), and policy applies the **`TeamRole` → `OrgAction`** matrix ([`org-action-policy-matrix.md`](./org-action-policy-matrix.md) §2).

- Governs **default** capabilities on **org-owned** CRM and **org-owned** workflow records (pipeline entities stored under the org).
- **`orgRoleAllows(role, action)`** answers: “May this role perform this org action **when** the resource is org-scoped and no harder deny applies?”

### 2.2 Scoped (relationship) access

**Scoped** access does **not** replace the org matrix; it **composes** with it ([`org-action-policy-matrix.md`](./org-action-policy-matrix.md) §3–4).

| Mechanism | What it gates | Typical use |
|-----------|----------------|---------------|
| **`participantPrincipalIds`** (and related workflow principals) | Record-level **participant** path on a workflow document | User added to a line item; may allow read/comment per workflow matrix even when org `viewer` has no org-wide workflow read |
| **Assignee fields** on requests (`assignedToUserIds` / `assignedToUserId`) | **Assignee** resource role for that request | Scheduling / “my work” without full org list access |
| **`tree_permissions`** | Per-tree collaborator role | External editor on a tree; may allow scoped tree writes **for that tree** even when org role is `viewer` (policy must define OR with org deny — see matrix §3.3) |
| **Personal / solo org** (`organizationId === uid`) | Treated as **org_member** for workflow resource derivation, not team `viewer` semantics | Pro solo pipeline |

**Resource role derivation (app):** [`lib/groundzy/policy/access/resource-role.ts`](../../../app/lib/groundzy/policy/access/resource-role.ts) (`deriveWorkflowResourceRoleSync`) — e.g. participant + `viewer` → **`participant`** row; participant + `member`+ in team scope → often **`org_member`**.

### 2.3 Evaluation order (summary)

1. Auth, Firestore rules hard denies, server hard denies, **tier entitlements** (e.g. pipeline, CRM modules).  
2. For org-scoped resources: **`orgRoleAllows`**.  
3. If org path denies but a **relationship grant** exists, evaluate **relationship / workflow matrix** per v3 access doc (see matrix §3.2–3.3).

---

## 3. CRUD by entity (`OrgAction` matrix)

Legend: **R** read, **C** create, **U** update, **D** delete. Values are **org-wide** defaults for org-owned records when tier entitlements already allow the surface.

### 3.1 Clients, properties, trees (CRM)

| Entity | Action | viewer | member | manager | admin | owner |
|--------|--------|--------|--------|---------|-------|-------|
| Trees | R | ✓ | ✓ | ✓ | ✓ | ✓ |
| Trees | C / U | ✗ | ✓ | ✓ | ✓ | ✓ |
| Trees | D | ✗ | ✗ | ✓ | ✓ | ✓ |
| Properties | R | ✓ | ✓ | ✓ | ✓ | ✓ |
| Properties | C / U | ✗ | ✓ | ✓ | ✓ | ✓ |
| Properties | D | ✗ | ✗ | ✓ | ✓ | ✓ |
| Clients | R | ✓ | ✓ | ✓ | ✓ | ✓ |
| Clients | C / U | ✗ | ✓ | ✓ | ✓ | ✓ |
| Clients | D | ✗ | ✗ | ✓ | ✓ | ✓ |

### 3.2 Requests, quotes, jobs, invoices (workflow)

For **each** of the four entities, **member / manager / admin / owner** share the same flags; **viewer** has **all ✗**.

| Entity | R / C / U / D | viewer | member | manager | admin | owner |
|--------|---------------|--------|--------|---------|-------|-------|
| Requests | each | ✗ | ✓ | ✓ | ✓ | ✓ |
| Quotes | each | ✗ | ✓ | ✓ | ✓ | ✓ |
| Jobs | each | ✗ | ✓ | ✓ | ✓ | ✓ |
| Invoices | each | ✗ | ✓ | ✓ | ✓ | ✓ |

**Normative:** **`member` and `manager` differ only on CRM delete** (and team/billing rows), **not** on workflow OrgActions — see matrix §2 notes.

### 3.3 Team & billing (org administration)

| OrgAction (summary) | viewer | member | manager | admin | owner |
|---------------------|--------|--------|---------|-------|-------|
| Invite / role update / remove members | ✗ | ✗ | ✗ | ✓ | ✓ |
| Team settings, invite codes | ✗ | ✗ | ✗ | ✓ | ✓ |
| Self-leave | ✓ | ✓ | ✓ | ✓ | ✗ (owner must transfer first) |
| Ownership transfer | ✗ | ✗ | ✗ | ✗ | ✓ |
| Billing manage | ✗ | ✗ | ✗ | ✗ | ✓ |

**Product rule (existing):** Even when `orgRoleAllows` allows admin to remove members, **admin-vs-admin** removal is blocked by an **extra** server check (matrix §4.4).

---

## 4. Sidebar & drawer visibility

### 4.1 Tier vs role

- **Subscription tier / entitlements** decide **whether** product modules exist (e.g. workflow pipeline drawers registered with `visibleForTiers` in [`lib/drawers.ts`](../../../app/lib/drawers.ts)).
- **`TeamRole`** decides **what an org member may do** inside those modules.

### 4.2 Current client behavior (workflow list drawers)

The **main org pipeline list** drawers **`requests`**, **`quotes`**, **`jobs`**, **`invoices`** are registered in the drawer system for Pro+ team tiers, but the **sidebar and bottom nav** additionally filter them:

- Implementation: [`components/navigation/sidebar.tsx`](../../../app/components/navigation/sidebar.tsx), [`components/navigation/bottom-nav.tsx`](../../../app/components/navigation/bottom-nav.tsx).
- Rule: hide those four ids unless **`workflowPerms.requests.read`** is true (aligned with matrix: all four `org.workflow.*.read` flags move together for known roles).
- Hook: [`hooks/useOrgWorkflowReadPermissions.ts`](../../../app/hooks/useOrgWorkflowReadPermissions.ts) (`ORG_WORKFLOW_LIST_DRAWER_IDS`, `useOrgWorkflowReadPermissions`).

**Effect:** **`viewer`** does **not** see those four list entries in the primary nav, because **`org.workflow.*.read`** is **false** for viewer in the matrix.

**List query gating:** Hooks use **`useWorkflowOrgListQueryEnabled`** (same file) — requires `organizationId`, **`workflow.pipeline`** entitlement, **and** per-entity **`perms[entity].read`**.

### 4.3 Participant / assignee surfaces (not the same as org lists)

- Drawer **`participant-work`** (“Work” in UI) lists workflow items where the user is a **participant** (and related server/API logic). It is **not** in `ORG_WORKFLOW_LIST_DRAWER_IDS`, so it is **not** removed by the same `requests.read` filter. **Tier** still restricts it via `visibleForTiers`.
- **Intended product model (v3):** A **`viewer`** may still have **record-scoped** workflow access when they are a **participant**, **assignee**, or similar — **without** receiving org-wide workflow read/create ([`org-action-policy-matrix.md`](./org-action-policy-matrix.md) §3.2). The **org list drawers** represent **org-wide** insider browsing; **participant-work** and **view-*** detail drawers represent **scoped** engagement.

### 4.4 Deep links & empty states

Workflow list drawer components (e.g. [`app/drawers/requests.tsx`](../../../app/app/drawers/requests.tsx)) may show a **role-denied** or upgrade state when entitlements allow the surface but **`workflowPerms.requests.read`** is false — so behavior is consistent even if navigation hid the entry.

---

## 5. Assignment & linking rules (product-level)

### 5.1 Assignees (requests)

- Requests can carry **assignee user id(s)** (see types / `getRequestAssigneeUserIds` in app).
- Policy uses assignee to derive **`assignee`** resource role where applicable ([`resource-role.ts`](../../../app/lib/groundzy/policy/access/resource-role.ts)), which can grant **scoped** access independent of browsing the full org request list.

### 5.2 Participants (all workflow entity types)

- **`participantPrincipalIds`** (principal-keyed) marks users as **participants** on a workflow document.
- **Viewer + participant:** resource role resolves to **`participant`** so workflow policy can use the **participant** row rather than treating them as full **org_member** on that record ([`resource-role.ts`](../../../app/lib/groundzy/policy/access/resource-role.ts) comments).

### 5.3 CRM linking / shared trees (teams)

- Team **`viewer`** semantics for **trees** can interact with **shared CRM links** and viewer org id helpers in the client (e.g. add-tree / map flows). That is **orthogonal** to **`OrgAction`** workflow rows: it concerns **how** tree and CRM data are **linked** across orgs, not the org-wide workflow matrix.
- **Normative:** Linking and sharing **compose** with org role and **`tree_permissions`**; they do not automatically grant org-wide workflow CRUD.

### 5.4 Workflow creates (global check)

- **Server/policy:** [`canCreateWorkflowEntity`](../../../app/lib/groundzy/policy/access/can.ts) requires **`workflowPipeline`** and, for team orgs, **`orgRoleAllows`** on the matching **`org.workflow.*.create`** action. **Viewer** fails the org-role check for creates.

---

## 6. Intended difference: **`member`** vs **`viewer`**

| Dimension | **Member** | **Viewer** |
|-----------|------------|------------|
| **Product intent** | Day-to-day operator who **mutates** org CRM and **runs** the org workflow pipeline (subject to tier). | **Read-only org insider** for **org-wide CRM**; **not** granted org-wide workflow **OrgAction**s. |
| **CRM** | Read + create + update; no delete until manager+. | **Read only** — no create/update/delete via org matrix. |
| **Org-wide workflow lists & CRUD** | All workflow **R/C/U/D** `OrgAction`s allowed in matrix (still subject to tier + server rules). | **None** in matrix — no org-wide pipeline read or write by profile alone. |
| **Participants / assignees / collaborators** | Still subject to relationship rules; often evaluated as **`org_member`** on team-scoped docs when in scope. | May use **participant** / **assignee** / **`tree_permissions`** paths for **scoped** access per v3; does **not** imply org-wide workflow browser rights. |
| **Team administration** | No (except self-leave). | No (except self-leave). |

**Summary sentence:** **`Member`** is the **default operational role** inside the org for CRM + workflow. **`Viewer`** is **read-only CRM at the org level** and **no org-wide workflow capabilities** in the `OrgAction` profile; **scoped** workflow or tree access is **additive** via relationships, not via the viewer org-wide matrix row.

---

## 7. Changing this spec

1. Update **[`org-action-policy-matrix.md`](./org-action-policy-matrix.md)** §1–2 for any new `OrgAction` or cell change.  
2. Update **`MATRIX`** in [`org-action.ts`](../../../app/lib/groundzy/policy/org-action.ts) to match **exactly**.  
3. Update UI shim and navigation filters if drawer visibility rules change.  
4. Update this document’s summary tables so product and engineering stay aligned.

---

## Related links

- [OrgAction policy matrix (authoritative table)](./org-action-policy-matrix.md)  
- [Organization roles & access (v3)](./organization-roles-and-access.md)  
- [Teams & roles — implementation audit](../../features/teams-and-roles-overview.md)
