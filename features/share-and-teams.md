# Share and Teams Features

## Share Feature

### Public Share Links

- **URL**: `/share/[token]`
- **Location**: `app/share/[token]/page.tsx`, `share-token-client.tsx`
- **Flow**:
  1. Unauthenticated user visits `/share/{token}`
  2. Redirected to sign-in with return URL `/share/{token}`
  3. After auth, `POST /api/share/{token}` with Bearer token
  4. Server resolves token → treeId, grants access via `shared_data`
  5. Client redirects to `/?drawer=view-tree&treeId={treeId}`

### Create Share Link

- **Location**: View Tree drawer → Share tab
- **Firestore**: `tree_share_links/{token}` – token = doc ID, fields: `treeId`, `createdBy`
- **API**: Share resolution is server-side (Firebase Admin)

### Grant Access by Username

- **Location**: View Tree → Share tab, Contact Us (share tree / approve access request), bulk share dialog
- **Flow**: Enter username → `getUserIdByUsername()` → **`POST /api/trees/[treeId]/grant-user`** with Bearer token and `{ targetUserId }`. The server uses the Firebase Admin SDK to write `tree_permissions/{treeId}/members/{userId}` and `user_tree_permissions/{userId}/trees/{treeId}` (client SDK cannot create these documents; rules deny direct writes).
- **Tree access requests**: User requests access → owner approves → same API grant + request status update

### Share with a Pro organization (team fan-out)

- **Location**: View Tree → Share tab (tree owner): “Share with my default pro’s team” when `users/{uid}.defaultProOrganizationId` is set; nearby certified pros with `hasGroundzyAccount` from `GET /api/groundzy-pros/nearby` show “Share with team” (resolves `groundzy_pros/{id}.linkedOrganizationId`).
- **API**: **`POST /api/trees/[treeId]/share-with-organization`** with Bearer token. Body: `{ organizationId?: string, certifiedProId?: string }` (provide one; if both, `organizationId` must match the pro’s `linkedOrganizationId`).
- **Behavior**: Loads member UIDs from `teams/{organizationId}`, grants each (except the caller) editor access with `grantSource: "organization"` and `sourceOrganizationId` on the permission doc; appends one access-history entry on the tree for the org share. **New team members after the share do not automatically gain access** until the owner shares again (MVP).

### Default certified pro (automatic sync)

- **Location**: Hire Pro → **Set as default pro** for a certified pro with a linked Groundzy business (`hasGroundzyAccount` + `linkedOrganizationId`). A **confirmation dialog** asks whether to share trees and contact details with the pro’s team; if the homeowner has **more than one property**, they can choose all properties, only the current property (when the drawer was opened with `propertyId` or `treeId`), or a custom subset.
- **API**: **`POST /api/user/sync-default-pro-share`** runs after the user confirms (Bearer token). With no `homeownerPropertyIds` in the body, it performs **per-tree org fan-out** for **all** of the homeowner’s owned trees (same scope as the tree list query). With **`homeownerPropertyIds`**, only those homeowner properties are mirrored to the pro org and only trees whose `propertyId` is in that set receive org-based grants (trees with no `propertyId` are excluded from partial scope). It **upserts** a Pro CRM **`clients/{gz_hm_{organizationId}_{homeownerUid}}`** document on the target org with `homeownerUserId`, contact fields, and a text summary of **property addresses** included in that sync (Pro orgs still cannot read `properties` under the homeowner’s uid via rules; this copies salient details into the client record).
- **Remove default pro**: Hire Pro → **Remove as default** calls **`POST /api/user/revoke-default-pro-share`** first (when `defaultProOrganizationId` was set), then clears default pro fields. Revoke deletes **org-based** tree permissions where `grantSource === "organization"` and `sourceOrganizationId` matches that business (same shape as share-with-organization / default sync). **Any** share to that same team using that mechanism is removed—not only the automatic sync. Updates the CRM stub with `defaultProRevokedAt` and a short note when the client doc exists.

### Shared Trees

- **Collection**: `shared_data/{treeId}` – `sharedWith: [userId1, userId2]`
- **Read access**: Firestore rules allow read if user in sharedWith
- **Restricted view**: `restricted-tree` drawer when user has shared access but not full ownership
- **Archive**: `ArchiveSharedTreeDialog` when tree is shared with others

### Bulk Share

- **Component**: `BulkShareDialog`
- **Flow**: Select multiple trees → share dialog → create links or grant access

---

## Teams Feature

### Overview

- **Tiers**: Small Team, Mid Team, Large Team, Enterprise
- **Collection**: Firestore `teams`, `team_members`, `invite_codes`
- **Roles**: owner, admin, manager, member

### Team Settings (`team-settings` drawer)

- **Location**: `app/drawers/team-settings.tsx`
- **Actions**: Invite members, manage roles, view team
- **Server actions**: `app/actions/team.ts`

### Invite Codes

- **Collection**: `invite_codes`
- **API**: `POST /api/invite-code/validate` – validate code, join team
- **Create**: Team owners/admins create invite codes

### Create Free Beta Team

- **API**: `POST /api/teams/create-free-beta`
- Creates team for beta users

### Organization Scoping

- **User doc**: `organizationId` = team ID for team members
- **Data**: Trees, clients, properties, zones, requests, quotes, jobs, invoices scoped by `organizationId`
- **databaseCode**: Individual Pro users use `databaseCode` instead of org

### Team Members

- **Collection**: `team_members` – links user to team
- **Team doc**: `members` map (uid → role), `memberRoles`, `ownerId`
- **Firestore rules**: `isInTeam()`, `isOwnerOrAdmin()`, `hasRole()`, `isTeamMember()`

### Access Control

| Role | Permissions |
|------|-------------|
| owner | Full access, delete team, manage invites |
| admin | Manage members, full data access |
| manager | Manage clients, properties, jobs; delete trees/clients/properties |
| member | Read/write own scope |
