# Unified permission model (v3) — locked decisions

**Status:** Normative (v3)  
**Purpose:** Single reference for architecture (reads vs writes), canonical role storage, org-role mental model, entity classification (`work_items` / `recurring_plans` vs CRM), permission evaluation examples, and collaborator UX language.  
**Audience:** Engineering, product, security, support.

**Related:** [Organization roles & access](./organization-roles-and-access.md) · [Permissions & access](./permissions.md) · [OrgAction policy matrix](./org-action-policy-matrix.md) · [Role capabilities (code-enforced)](../../features/role-capabilities-matrix.md) · Code: [`lib/groundzy/policy/org-action.ts`](../../../app/lib/groundzy/policy/org-action.ts).

---

## 1. Reads vs writes (not flexible)

| Layer | Responsibility |
|-------|----------------|
| **Reads** | **Firestore security rules** enforce a **coarse** outer boundary: membership (`organizationId`), participant index (`participantPrincipalIds`), `databaseCode` / solo paths, active `tree_permissions`, etc. **Server loaders** (Admin SDK + policy) supply **authoritative** access envelopes for sensitive UX (e.g. workflow entity views). |
| **Writes** | **Server-authoritative** for mutations that depend on **composition** of org role + tree grant + workflow participation + entitlements. Policy is evaluated in TypeScript (`orgRoleAllows`, `canOrgOrTreeAction`, workflow gates). Client direct Firestore **mutations** on org-scoped / shared / workflow data are **progressively narrowed or denied**; rules remain a **safety net**, not a full duplicate of composed logic. |

**Rationale:** Composition does not map cleanly to Security Rules; duplicating every `OrgAction` row in rules causes **rule drift**. The primary strategy is **not** to fully replicate matrix logic in rules for complex paths.

---

## 2. Canonical team role (hard requirement)

| Storage | Role |
|---------|------|
| **`teams/{teamId}.memberRoles[uid]`** | **Only source of truth** for `TeamRole` in that org. |
| **`users/{uid}.role`** | **Derived cache** for Firestore helpers (`hasRole`, `isOwnerOrAdmin`) and UI. **Not** a second opinion. |

**Enforcement:**

- Every membership or role change **updates both** in **one transaction** (see [`app/actions/team.ts`](../../../app/app/actions/team.ts)).
- **Drift** vs canonical is a **data bug**; repair from `teams.memberRoles`. Optional **background repair job** may sync stale user docs when telemetry indicates mismatch.
- **Logging:** [`logTeamMemberRoleDriftIfAny`](../../../app/lib/groundzy/policy/team-role-invariant.ts) when team data is available on read paths.
- **Server:** Prefer loading canonical role from **`teams`** when the team document is already in context; use `users.role` only as cache when the team is not loaded.

**Long-term:** Rules might read less from `users.role` if cost/complexity tradeoffs allow; until then the cache **must** match canonical on every write.

---

## 3. Org role mental model (product language)

Normative meanings align with [`org-action.ts`](../../../app/lib/groundzy/policy/org-action.ts) (`orgRoleAllows`).

| Role | Meaning (org-scoped default) |
|------|------------------------------|
| **Viewer** | Read-oriented on org CRM/workflow per matrix; cannot rely on org membership alone to mutate org-scoped CRM/workflow where the matrix denies it. |
| **Member** | Create/update org CRM and workflow entities per `OrgAction`; cannot delete CRM entities or perform team administration. |
| **Manager** | Member + **delete** on CRM entities (trees/properties/clients) where the matrix grants manager+. |
| **Admin** | Manager + **team administration** (invites, role changes, settings). **Not** billing or **org** ownership transfer (`org.billing.manage`, `org.team.ownership.transfer` are **owner-only** in the matrix). Firestore **`isOwnerOrAdmin`** uses `users.role` **owner \| admin** for **team doc / invite_code** paths — team admin only, not billing. |
| **Owner** | Admin + **billing** + **ownership transfer** (matrix). |

**Surprising but correct (must match product copy to code):**

- Some **tree API** routes gate team-scoped callers with **`org.tree.update`** — **member+** may qualify under the matrix and routes (e.g. transfer prerequisites). That is **broader** than informal “member = CRM only”; document **actual** capabilities.
- **`Admin`** is fully specified in **`OrgAction`** for team ops; exclude **billing** and **org ownership transfer** from admin’s row.

---

## 4. Entity classification: CRM vs system indices

Use this to explain **different delete/update thresholds** in rules and policy.

| Class | Examples | Destructive / high-impact threshold |
|-------|-----------|-------------------------------------|
| **CRM-class** | `clients`, `properties`, `trees` (org-scoped team path) | **Manager+** deletes (`teamOrgMutateManagerPlus` pattern in rules). |
| **System-class** | **`work_items`** delete, **`recurring_plans`** delete | **Owner + Admin only** in Firestore rules — org-wide operational control, not day-to-day field CRM. |

**work_items / recurring_plans** are **system / scheduling indices**, not the same CRM record class as clients/properties/trees for destructive policy.

**Recurring plan updates:** Rules currently allow **`isInTeam`** without `users.role` granularity — **migrate** toward server-validated updates with `orgRoleAllows` where appropriate, or document intentional coarse rules until migration completes.

---

## 5. Collaborator vs org role (UX)

Collaborator grants **`tree_permissions`** **compose** with org role ([`canOrgOrTreeAction`](../../../app/lib/groundzy/policy/tree-collaborator.ts)); they are **not** a separate permission product.

**Required UI copy patterns:**

- Always show **Org: &lt;TeamRole&gt;** when displaying team membership context.
- When `tree_permissions` applies, show **On this tree: &lt;grant role&gt;** (or equivalent).
- **Never** show a single ambiguous label **“Viewer”** when **both** org role and tree grant apply — users will assume one global role.

See [Collaborator & org role — UX guidelines](../../features/collaborator-org-role-ux.md).

---

## 6. Permission evaluation examples

These examples are **normative for explanation** (support/product). Engineering **must** keep them aligned with [`evaluateWorkflowDocumentRead`](../../../app/lib/server/workflow-document-read.ts), [`teamOrgWorkflowReadDenied`](../../../app/lib/groundzy/policy/org-action.ts), [`canOrgOrTreeAction`](../../../app/lib/groundzy/policy/tree-collaborator.ts), and [`org-action`](../../../app/lib/groundzy/policy/org-action.ts). Update this section in the **same PR** as policy changes.

**Example 1 — Org viewer, tree editor (collaborator)**

- Org role: **Viewer**. Tree grant: **Editor** (active `tree_permissions`).
- **Update tree** (scoped fields covered by collaborator + rules)? **Yes** — collaborator path can allow updates where org role alone would not; see `canOrgOrTreeAction` + Firestore tree update rules for active collaborators.

**Example 2 — Org viewer, not workflow participant**

- Org role: **Viewer**. Workflow doc is team-org-owned; user **not** in `participantPrincipalIds`; no solo/dbCode bypass.
- **Read workflow** (server/product path)? **No** — `teamOrgWorkflowReadDenied` denies viewer for team-org workflow reads when the only tie is same-org membership (`evaluateWorkflowDocumentRead`).

**Example 2b — Org member, not participant (same org)**

- Org role: **Member**. Same-org workflow doc; **not** a participant.
- **Read?** **Yes** (current implementation) — matrix grants `org.workflow.*.read` for member; server gate allows same-org read when not denied by `teamOrgWorkflowReadDenied`. Changing this would be an explicit product/policy change.

**Example 3 — Org viewer, workflow participant**

- Org role: **Viewer**. `participantPrincipalIds` contains `uid:{uid}`.
- **Read workflow?** **Yes** — participant branch is evaluated **first** in `evaluateWorkflowDocumentRead`; viewer org read denial does not apply to participants.

---

## Related links

- [Implementation plan: unified `can()` API surface](./can-api-implementation-plan.md)
- [Teams & roles — implementation audit](../../features/teams-and-roles-overview.md)
- [Groundzy v3 — Access & Permission System](../../architecture/Groundzy%20v3%20—%20Access%20%26%20Permission%20System.md)
