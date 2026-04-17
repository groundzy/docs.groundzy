# Role capabilities (code-enforced)

Grounded in `firebase/firestore.rules`, `lib/groundzy/policy/**`, `lib/permissions-utils.ts`, `app/actions/team.ts`, and `app/api/trees/**`. Do not treat this as exhaustive for routes not traced here.

**v3 posture:** [Unified permission model (v3)](../groundzy-v3-docs/05-data/unified-permission-model-v3.md) · [Collaborator & org role UX](./collaborator-org-role-ux.md).

## v3 architecture mapping (reads vs writes)

How each area aligns with [unified-permission-model-v3.md §1](../groundzy-v3-docs/05-data/unified-permission-model-v3.md):

| Area | Firestore reads | Firestore writes / mutations | Authoritative policy |
|------|-----------------|------------------------------|----------------------|
| CRM (trees, clients, properties), zones | Coarse membership + Phase D `hasRole` helpers | Same helpers on team paths; progressive **narrow/deny** client writes per migration | `orgRoleAllows`, `teamOrgMutate*` in rules |
| Workflow entities | `workflowTeamAccess` / participant / solo paths | **Prefer server/API** for composed checks; rules broader than matrix for some client paths — **server is truth** for product | `evaluateWorkflowDocumentRead`, `canCreateWorkflowEntity`, `compute-*-Access` |
| `work_items`, `recurring_plans` | Team / dbCode / shared-tree paths | **System-class** deletes (owner/admin in rules); recurring **updates** coarse in rules — migrate to server + matrix over time | [§4 Entity classification](../groundzy-v3-docs/05-data/unified-permission-model-v3.md#4-entity-classification-crm-vs-system-indices) |
| Tree share grants, archive, transfer | Tree + org membership + collaborator | **API routes** + `assertOrgRoleAllows` / `canOrgOrTreeAction` | Same modules |
| Team admin, invites | Team membership reads | Server actions `verifyOwnerOrAdmin` + transactional role updates | `teams.memberRoles` canonical; `users.role` cache |

## Org & collaborator capabilities

| Action / Capability | Viewer | Member | Manager | Owner | Notes |
|--------------------|:------:|:------:|:-------:|:-----:|-------|
| **CRM — read** trees / clients / properties (team-scoped org) | ✓ | ✓ | ✓ | ✓ | `orgRoleAllows` reads = true for all (`org-action.ts`). Firestore allows read when `isInTeam(organizationId)` (or solo/dbCode/home paths). |
| **CRM — create / update** trees / clients / properties (team-scoped org) | ✗ | ✓ | ✓ | ✓ | `teamOrgMutateMemberPlus` requires `users.role ∈ {member, manager, admin, owner}`; aligns with `orgRoleAllows` for create/update. |
| **CRM — delete** clients / properties / trees (team-scoped org) | ✗ | ✗ | ✓ | ✓ | `teamOrgMutateManagerPlus` + matrix: delete only manager+. |
| **Zones — create / update** (team-scoped org) | ✗ | ✓ | ✓ | ✓ | Same as CRM create/update (`teamOrgMutateMemberPlus`). Zone **hard delete** denied for everyone in rules (`allow delete: if false`). |
| **Zone services — create / update / read** | ✗ | ✓ | ✓ | ✓ | Uses `teamOrgMutateMemberPlus` on parent zone org. |
| **Workflow — read via app/server** (same team org; not a participant) | ✗ | ✓ | ✓ | ✓ | `evaluateWorkflowDocumentRead` + `teamOrgWorkflowReadDenied`: viewer denied for team org docs; matrix marks all `org.workflow.*.read` false for viewer. |
| **Workflow — read if `participantPrincipalIds` includes user** | ✓ | ✓ | ✓ | ✓ | Checked **first** in `evaluateWorkflowDocumentRead`; participant path does not apply viewer denial. Firestore allows read if `participantPrincipalIds` contains `uid:{uid}` **or** `workflowDocNonParticipantRead()`. |
| **Workflow — org_member mutate flags** (approve/update/etc. from `compute-*-Access`) | ✗ | ✓ | ✓ | ✓ | On **team-owned** docs, mutations gated by `orgRoleAllows(..., org.workflow.*.update)`. Viewer stays read/comment-only per resource-role wiring. |
| **Workflow — create via commands** (`canCreateWorkflowEntity`) | ✗ | ✓ | ✓ | ✓ | Requires `workflowPipeline` entitlement + `orgRoleAllows` for the entity create action (team org). |
| **Firestore client: workflow create/update/delete** (team org on doc) | ✗† | ✓ | ✓ | ✓ | †Rules use `workflowTeamAccess` **without** org-role checks on mutate/delete and org match on create—**broader than** matrix for viewers if clients write Firestore directly; product/server paths use matrix + gates above. Mark **policy vs rules mismatch** for viewer writes. |
| **Recurring plans — update** (team-scoped) | ✓ | ✓ | ✓ | ✓ | Rules: `isInTeam` only—**no** `users.role` gate (differs from matrix-only mental model). |
| **Recurring plans — delete** | ✗ | ✗ | ✗ | ✓\* | Rules: `isInTeam && hasRole(['owner','admin'])`; solo/dbCode/home paths unchanged. \*Treat **Admin** like Owner here; Manager/Member ✗. |
| **Work items — read / update** (team org on doc) | ✓ | ✓ | ✓ | ✓ | Rules: `isInTeam` without role distinction. |
| **Work items — delete** (team org on doc) | ✗ | ✗ | ✗ | ✓\* | Rules: `isInTeam && hasRole(['owner','admin'])`. \*Admin ✓; Manager/Member/Viewer ✗. |
| **Trees — grant shares / POST permissions / sharing grants** (`org.tree.update`) | ✗ | ✓ | ✓ | ✓ | After ownership checks, team-scoped callers must pass `assertOrgRoleAllows(..., "org.tree.update")` (`permissions` route, `tree-sharing-grant`). |
| **Trees — archive (soft-delete) API** | ✗‡ | ✗‡ | ✓‡ | ✓‡ | ‡`canOrgOrTreeAction`: **org** viewer/member cannot delete team tree; **manager+** can. Active **collaborators** may delete if grant role is owner/manager/editor (**not** contributor/viewer per `treeCollaboratorRoleAllowsArchive`). **Personal/solo bucket** shortcuts `implicitBucketOwner`. |
| **Trees — transfer ownership API** | ✗ | ✓ | ✓ | ✓ | Requires tree org = caller org + `assertOrgRoleAllows(..., "org.tree.update")`; not limited to Owner role alone—member+ can qualify if caller is actual tree-owning principal per route checks. |
| **Invite codes — create / revoke / regenerate** | ✗ | ✗ | ✗ | ✓\* | Firestore: `isOwnerOrAdmin` = `users.role ∈ {owner, admin}`. Server actions: `teams.ownerId` OR `teams.memberRoles[userId] === 'admin'` (`verifyOwnerOrAdmin`). \*Also **Admin** org role (**not** Manager). |
| **Team — update member roles / remove members / extra invites** | ✗ | ✗ | ✗ | ✓\* | Same as row above: owner or **memberRoles admin** in `app/actions/team.ts`; admin peer restrictions in `team-admin-peer-rule.ts`. |
| **Team — transfer ownership / billing** (`org.team.ownership.transfer`, `org.billing.manage`) | ✗ | ✗ | ✗ | ✓ | Matrix: **Owner** only—**Admin** ✗ on these two actions (`org-action.ts`). **Unclear** whether separate routes enforce transfer/billing beyond Stripe/matrix—Stripe mapping exists but full enforcement not traced here. |
| **Team — self-leave** | ✓ | ✓ | ✓ | ✗ | Matrix: owner cannot (`org.team.membership.self_leave`). |

**Collaborator grants** (`tree_permissions.members.{uid}.role`; not org-role columns): **viewer** → read-oriented tree/media/timeline access where rules reference active collaborator; **contributor** → cannot satisfy archive-as-collaborator delete path; **editor / manager / grant “owner”** → archive path allowed per `treeCollaboratorRoleAllowsArchive` / update rules.

**Admin column omitted from table:** wherever Owner is ✓ for team/invite/billing-adjacent rows, treat **Admin** like Owner **except** matrix-explicit Owner-only rows (ownership transfer, billing manage) and Firestore deletes that list only `owner` + `admin`.

## Key rules observed

- **`users.role`** (parsed by `parseTeamRoleFromUserDoc`) drives `hasRole()` in Firestore helpers (`teamOrgMutateMemberPlus` / `ManagerPlus`) and `orgRoleAllows`; **viewer cannot org-write** CRM/tree counters/zones/org CRM even when in the team—**membership alone does not upgrade viewer**.
- **Org policy matrix** in `org-action.ts` matches Phase D Firestore helpers for CRM/tree deletes and team **invite/matrix** semantics; **Admin** matches Owner on most rows **except** `org.team.ownership.transfer` and `org.billing.manage` (both Owner-only there).
- **Workflow**: server read path **blocks viewer** from non-participant team-org reads (`teamOrgWorkflowReadDenied`), while **participant PrincipalIds** bypass that; **Firestore** non-participant reads remain **role-agnostic** (`workflowTeamAccess`)—**server/product vs raw client rules differ** for viewer.
- **Team admin operations** use **`teams.memberRoles`** (owner or admin) in server actions; **Firestore `teams` update** also accepts `users.role` owner/admin or `memberRoles[uid]=='admin'`—multiple checks can apply.
- **Shared trees**: **`canOrgOrTreeAction`** combines **team org role matrix** with **active `tree_permissions`** grant (explicit collaborator roles gate destructive/update behavior when org role alone would fail).
- **`work_items` delete** and **`recurring_plans` delete** are **owner/admin-only** in rules, unlike CRM delete (manager+)—**role thresholds differ by collection**.
