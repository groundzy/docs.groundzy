# Teams & team roles — implementation and plans

This document summarizes **what exists today** in the Groundzy app for **teams** (org subscriptions) and **team roles**, and how that lines up with **architecture docs** and **known gaps**. It is derived from the codebase under `C:\Groundzy\app` and docs under `C:\Groundzy\docs` (as of the analysis date).

**→ v3 (normative):** [Organization roles & access](../groundzy-v3-docs/05-data/organization-roles-and-access.md) → [OrgAction policy matrix](../groundzy-v3-docs/05-data/org-action-policy-matrix.md) (implementable `OrgAction` contract).

---

## 1. What we have: product surface

| Area | What exists |
|------|-------------|
| **Tiers** | Small Team, Mid Team, Large Team, Enterprise — seat caps come from `@groundzy/pricing` (`TEAM_BAND_MAX_MEMBERS`, `ENTERPRISE_MAX_MEMBERS` in `packages/pricing/src/team-seats.ts`). |
| **Team document** | Firestore `teams/{teamId}`: `name`, `ownerId`, `ownerEmail`, `members` (UID array), `memberRoles` (UID → role), `settings`, `tier`, subscription fields, `logoURL`, flags like `isActive` / `isDeleted`. |
| **User membership** | `users/{uid}.organizationId` = team id; `users/{uid}.role` = the user’s **canonical** `TeamRole` (kept in sync with `teams.memberRoles` for that user). |
| **Invites** | `invite_codes` collection; server actions create/revoke/regenerate codes (`app/actions/team.ts`). **`POST /api/invite-code/validate`** validates codes for joining. |
| **Beta / ops** | **`POST /api/teams/create-free-beta`** creates teams for beta flows (referenced in internal docs). |
| **Stripe** | Team creation from checkout uses `createTeam` / `createTeamForWebhook` (`app/actions/team.ts`) after subscription; team links to Stripe via `subscriptionId`, `subscriptionStatus`, etc. |
| **UI** | Organization UX lives in **`TeamSettingsPanel`** (`app/drawers/team-settings/TeamSettingsPanel.tsx`) — invites, role changes, removal, leave, ownership transfer, logo upload, workflow defaults via **`useWorkflowSettings`** / **`WorkflowSettingsEditor`**. The legacy **`team-settings`** drawer wraps the same panel (`app/drawers/team-settings.tsx`); docs note preferring **Settings → Organization** (`tab=organization`). |
| **Sharing (related)** | Teams are the **organization** behind “share with org” / default-pro flows; that is **tree-level** sharing (`tree_permissions`), not the same as team roles — see `docs/features/share-and-teams.md` and `groundzy-v3-docs/06-features/teams-and-sharing.md`. |

---

## 2. Canonical roles (`TeamRole`)

**Source of truth (types):** `app/types/team.ts`

```ts
export type TeamRole = 'owner' | 'admin' | 'manager' | 'member' | 'viewer';
```

### Signup-time mapping

Signup collects **`SignupUserRole`**: `owner | admin | manager | staff | other`. **`signupRoleToCanonical`** maps:

| Signup | Canonical `TeamRole` |
|--------|-------------------------|
| owner | owner |
| admin | admin |
| manager | manager |
| staff | member |
| other | viewer |

Used when creating a team (`createTeam` / `createTeamForWebhook`) so the creator’s `memberRoles` and `users.role` are consistent.

### Joining via invite default

`joinTeam` resolves the new member’s role with **`resolveJoinRole`** (`app/actions/team.ts`):

- If `defaultRole` is passed (`member` \| `viewer`), that wins.
- Else if `team.settings.defaultPermissions === 'view'` → **`viewer`**.
- Else → **`member`**.

New teams set `settings.defaultPermissions` to **`'view'`** at creation, so **invite joins default to viewer** unless the flow passes `defaultRole` or settings are changed elsewhere.

---

## 3. Role permissions in the app (UI / client policy helpers)

**File:** `app/lib/permissions-utils.ts`

This module defines **`RolePermissions`** (trees, properties, clients, jobs, quotes, invoices, requests CRUD flags plus `inviteMembers`, `changeRoles`, `removeMembers`, `teamSettings`, `billing`) and maps each **`TeamRole`** to a matrix.

**Summary:**

| Role | Elevated behavior vs previous row |
|------|-------------------------------------|
| **viewer** | Read trees, properties, clients; no write on those entities; no jobs/quotes/invoices/requests; no team admin actions. |
| **member** | Create/update (not delete) trees, properties, clients; full workflow entity R/W for jobs, quotes, invoices, requests. |
| **manager** | Adds **delete** on trees, properties, clients. |
| **admin** | Adds **inviteMembers**, **changeRoles**, **removeMembers**, **teamSettings**. |
| **owner** | Adds **billing** (all admin capabilities + billing). |

**Important:** `getPermissionsForRole` returns **owner-equivalent permissions** when role is missing or unknown — intended for **non-team / individual** users, not a safe default for ambiguous team cases.

---

## 4. Server actions (authoritative mutations)

**File:** `app/actions/team.ts` (unless noted)

| Action | Who can run | Notes |
|--------|--------------|------|
| `createTeam` / `createTeamForWebhook` | Authenticated purchaser / webhook | Seeds team + user `organizationId` / `role`. |
| `joinTeam` | Invited user | Respects seat cap; updates `members`, `memberRoles`, user `organizationId`, `role`, subscription tier fields. |
| `updateMemberRole` | Owner or **admin** | Target roles: **`admin` \| `manager` \| `member` \| `viewer`** only — not `owner`. Cannot change owner’s role. **Admins** cannot promote/demote **admins** (restriction in code). |
| `removeMember` | Owner or admin | Cannot remove owner. |
| `leaveTeam` | Any non-owner member | Owner must transfer first. |
| `transferOwnership` | Owner only | New owner becomes `owner`; previous owner becomes **`admin`** in both team and user docs. |
| Invite code ops | Owner or admin | `createTeamInviteCode`, `revokeInviteCode`, `regeneratePrimaryInviteCode` |

---

## 5. Firestore security rules vs app roles

**File:** `app/firebase/firestore.rules`

- **`isInTeam(teamId)`** uses **`users.organizationId == teamId`** — any org member passes, **not** “role at or above X”.
- **`isOwnerOrAdmin(teamId)`** = in team **and** `user.role` in **`['owner','admin']`**.

Team document **updates** are allowed for `isOwnerOrAdmin` **or** team `ownerId` **or** `memberRoles[uid] == 'admin'` (redundant paths for edge cases).

**Implication:** **Manager**, **member**, and **viewer** are **not** treated as elevated in rules the way **owner/admin** are. Finer CRM/workflow constraints for those roles rely on **app logic** (`permissions-utils`, server routes, and the v3 access work) and on **tree / work_item** rules — not on a distinct “manager” branch in `firestore.rules` for the `teams` collection.

---

## 6. Documentation: what matches and what drifts

| Doc | Alignment |
|-----|-----------|
| `docs/groundzy-v3-docs/05-data/permissions.md` | States **`TeamRole`** includes **viewer**; notes **staff → member** mapping; flags **user.role** vs **team.memberRoles** as two sources to keep consistent. |
| `docs/features/share-and-teams.md` | Lists roles as **owner, admin, manager, member** — **omits viewer** (drift vs `TeamRole`). |
| `docs/architecture/Groundzy v3 — Access & Permission System.md` | **`AccessActor.orgMemberships[].role`** only lists **`owner \| admin \| manager \| member`** — **no viewer** in the typed actor model (planned taxonomy gap vs product types). |
| `docs/architecture/visibility-permission-model-v2.md` | Describes **team role inheritance** vs **tree_permission** precedence for trees — orthogonal but related to org roles. |
| `.cursor/plans/v3_unified_access_model_*.plan.md` | **Unified access model** implementation plan (many todos **completed**): policy **`can(actor, action, resource)`**, server-authoritative access for workflow views, **`tier ≠ access`** — complements but does not replace **team membership roles**. |

---

## 7. Planned / architectural direction (from docs)

1. **Single permission story:** v3 docs push **`Actor + Resource + Action → Policy`** and warn that **UI must not invent access logic** (`Groundzy v3 — Access & Permission System.md`).
2. **Unify role sources:** Resolve tension between **`users.role`** and **`teams.memberRoles`** explicitly in engineering guidelines (`permissions.md` “Inconsistencies to resolve”).
3. **`viewer` in `AccessActor`:** The canonical access doc now includes **`viewer`** in **`orgMemberships.role`** — implement policy and loaders accordingly ([`Groundzy v3 — Access & Permission System.md`](../architecture/Groundzy%20v3%20—%20Access%20%26%20Permission%20System.md) §3).
4. **Firestore vs UI:** “Finer roles may be partially enforced in app UI” (`permissions.md`) — closing that gap means either **rules** that understand manager/member/viewer for sensitive collections or **server-only** reads/writes (already direction for workflow entities in the unified access plan).

---

## 8. Quick file index

| Topic | Location |
|-------|-----------|
| Types & signup mapping | `app/types/team.ts` |
| Client role matrix | `app/lib/permissions-utils.ts` |
| Team mutations | `app/actions/team.ts` |
| Rules | `app/firebase/firestore.rules` (`isInTeam`, `isOwnerOrAdmin`, `teams`, `team_members`) |
| Org UI | `app/drawers/team-settings/TeamSettingsPanel.tsx` |
| Seat limits | `packages/pricing/src/team-seats.ts` |
| v3 permissions overview | `docs/groundzy-v3-docs/05-data/permissions.md` |
| Teams + sharing feature blurb | `docs/groundzy-v3-docs/06-features/teams-and-sharing.md` |

---

## 9. Summary

**Implemented today:** Five **canonical team roles** (`owner` through `viewer`), dual storage on **`teams.memberRoles`** and **`users.role`**, invite-based joins with **default viewer** when `defaultPermissions === 'view'`, a detailed **UI permission matrix**, and **server actions** for membership and ownership. **Firestore** broadly distinguishes **org membership** from **owner/admin** elevation; **manager/member/viewer** distinctions are primarily **application-layer**.

**Plans / docs:** The **v3 access model** pushes **policy-based**, **server-validated** access and separation of **tier** from **record access**; **team roles** remain the org-side identity layer but should stay aligned with **`AccessActor`** and with **resolved inconsistencies** (`user.role` vs `memberRoles`, **viewer** in canonical types vs actor model).
