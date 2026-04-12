# Tree Visibility, Privacy & Sharing — In-Depth Technical Documentation

This document explains how tree visibility, privacy, and sharing work in the Groundzy app, including Firestore rules, data flow, and UI behavior.

---

## Table of Contents

1. [Overview](#overview)
2. [Access Control Model](#access-control-model)
3. [Firestore Collections & Rules](#firestore-collections--rules)
4. [Visibility Layers](#visibility-layers)
5. [Sharing Mechanisms](#sharing-mechanisms)
6. [Archive & Ownership Transfer](#archive--ownership-transfer)
7. [Data Flow Diagrams](#data-flow-diagrams)
8. [Key Code Locations](#key-code-locations)
9. [Planned Upgrade (Migration Plan)](#planned-upgrade-migration-plan)

---

## Overview

Tree visibility and privacy are enforced at multiple layers:

| Layer | Purpose |
|-------|---------|
| **Firestore rules** | Server-side enforcement; determines who can read/write trees |
| **tree_public** / **tree_public_summaries** | Limited public info for map markers (any authenticated user); `tree_public` is the preferred map projection |
| **tree_permissions** / **user_tree_permissions** | Per-user share access to full tree data (`tree_permissions/{treeId}/members/{uid}`). Legacy **`shared_data`** is deprecated and not used by current rules |
| **tree_share_links** | External share links (redeem → **server** grants `tree_permissions` + `user_tree_permissions`) |
| **tree_access_requests** | Request/approve flow for access |
| **archivedByOwnerAt** | Owner-only archive (shared users keep access) |

---

## Access Control Model

### Who Can Access a Tree?

A user can **read** a tree document if **any** of these is true:

1. **Global admin** — User is in `admins/{uid}`
2. **Team member** — Tree's `organizationId` matches user's team (`isInTeam`)
3. **Database owner** — Tree's `databaseCode` matches user's `databaseCode` (Pro/Home tier)
4. **Legacy/home-tier** — Tree's `organizationId` equals user's `uid` (parity with **update** rules so personal-scope trees stay readable after org/team changes when the tree is still keyed by uid)
5. **Shared access** — Active row in `tree_permissions/{treeId}/members/{uid}` with `status == 'active'` (and mirrored `user_tree_permissions`)

### Ownership Concepts

- **Tree owner**: The `organizationId` (team) or `databaseCode` (individual) that owns the tree
- **Shared user**: User with an active `tree_permissions` member doc — has read/update access but not ownership
- **Owner-archived**: Tree has `archivedByOwnerAt` set; owner sees it in "Archived by me", shared users still have full access

---

## Firestore Collections & Rules

### trees/{treeId}

**Read** — allowed if:
- `isGlobalAdmin()` OR
- Tree has no `organizationId` / null / empty OR `isInTeam(organizationId)` OR
- `databaseCode == getUserDatabaseCode()` OR
- `resource.data.organizationId == request.auth.uid` (legacy home / personal scope) OR
- Active `tree_permissions/{treeId}/members/{request.auth.uid}` with `status == 'active'`

**Create** — caller must own the tree (org/team/databaseCode).

**Update** — team, `databaseCode`, legacy `organizationId == uid`, or active `tree_permissions` (see `firebase/firestore.rules`).

**Delete** — `false` (soft delete via `isDeleted` only).

### shared_data/{treeId} (legacy)

Older deployments may still have documents here. **Current Firestore rules use `tree_permissions`**, not `shared_data`, for collaborative access. Prefer `grantTreePermission` / share-link APIs which write `tree_permissions` and `user_tree_permissions`.

### tree_public_summaries/{summaryId}

`summaryId` = `treeId`. Contains limited fields: `gzTin`, `commonName`, `height`, `lat`, `lng`, `organizationId`, etc.

- **Read**: All authenticated users
- **Create/Update**: Tree owner only (must own corresponding tree)
- **Delete**: `false` (soft delete via `isDeleted`)

Synced from trees by owners via `upsertTreePublicSummary()`. Shared users do **not** write summaries (they lack write permission).

### tree_share_links/{token}

Token = document ID (opaque). Fields: `treeId`, `createdBy`, `revoked`, `expiresAt`.

- **Create**: Tree owner only
- **Read/Update**: Tree owner only (list links, revoke)
- **Delete**: `false`

Redeem is **server-side only** via `POST /api/share/[token]`.

### tree_access_requests/{requestId}

Fields: `treeId`, `gzTin`, `requesterId`, `ownerOrganizationId`, `status` (pending/approved/rejected), `message`, etc.

- **Read**: Requester OR owner (organization)
- **Create**: Authenticated user for themselves, `status == 'pending'`
- **Update**: Owner only (approve/reject)
- **Delete**: `false`

---

## Visibility Layers

### 1. Own Trees (Full Access)

**Source**: `listTrees(organizationId, { databaseCode?, excludeOwnerArchived })` + `getTreesSharedWithUser(userId)` merged in `useTrees`.

**Map**: Rendered as `TreeMarker` (own trees). Click → `view-tree` drawer.

**Exclusions**:
- `isDeleted === true` — not returned
- `archivedByOwnerAt` — excluded from main list when `excludeOwnerArchived: true`; shown in "Archived by me" via `listTreesArchivedByOwner`

### 2. Shared Trees (Full Access)

**Source**: `getTreesSharedWithUser(userId)` → resolves tree IDs from `user_tree_permissions/{userId}/trees`, then fetches each tree via `getTree`.

**Map**: Same as own trees — rendered as `TreeMarker` with `isShared={true}`. Click → `view-tree`.

**Merge**: `useTrees` merges shared trees into the main list so they appear alongside own trees.

### 3. Unrestricted / Public Summaries (Limited View)

**Source**: `listTreePublicSummaries()` → all non-deleted summaries. Filtered client-side to exclude own tree IDs.

**Map**: Rendered as `RestrictedTreeMarker` at zoom ≥ 14 (public) or ≥ 18 (private). Click → `restricted-tree` drawer.

**Data**: `TreePublicSummary` — species, height, location, GZ-TIN. No health, history, or sensitive data.

**Backfill**: `map-markers.tsx` calls `upsertTreePublicSummary(tree)` for own trees (skips shared trees — no write permission).

---

## Sharing Mechanisms

### 1. Share Link (External)

**Flow**:
1. Owner creates link: `createTreeShareLink(treeId, userId)` → writes `tree_share_links/{token}`
2. Recipient visits `/share/{token}`
3. If not signed in → redirect to sign-in with return URL
4. After auth, client calls `POST /api/share/{token}` with `Authorization: Bearer <idToken>`
5. Server validates token, fetches link, checks `revoked` and `expiresAt`
6. Server grants `tree_permissions` + `user_tree_permissions` for the recipient
7. Server appends access history entry to tree
8. Response: `{ treeId }`
9. Client redirects to `/?drawer=view-tree&treeId={treeId}`

**API**: `app/api/share/[token]/route.ts` — uses Firebase Admin SDK.

### 2. Grant by Username (Direct)

**Flow**:
1. Owner enters username in View Tree → Share tab
2. `getUserIdByUsername(username)` → `userId`
3. `grantTreeAccess` / permission APIs → `tree_permissions` + `user_tree_permissions`
4. `addTreeAccessHistoryEntry(treeId, userId, ownerId)` → audit trail in Activity tab

**Location**: `app/drawers/view-tree/index.tsx`, `handleShareWithUser`

### 3. Access Request (Request → Approve)

**Flow**:
1. User sees restricted tree on map (public summary), clicks → `restricted-tree` drawer
2. User clicks "Request Access" → `createTreeAccessRequest(treeId, gzTin, requesterId, ownerOrganizationId, message?)`
3. Request stored in `tree_access_requests` with `status: 'pending'`
4. Owner sees requests in Contact Us / access requests inbox
5. Owner approves → grant tree permission for `requesterId` + `updateTreeAccessRequestStatus(requestId, 'approved', userId)`
6. Requester's `useTrees` refetches (staleTime 1 min) → tree appears in list; `restricted-tree` redirects to `view-tree` when `fullTree` loads

**Location**: `app/drawers/restricted-tree.tsx`, `app/drawers/contact-us.tsx`

---

## Archive & Ownership Transfer

### Archive for Everyone

- Soft delete: `isDeleted: true`, `deletedAt`, `deletedBy`
- Public summary: `isDeleted: true`, `deletedAt` (via `POST /api/trees/archive` for shared users who can't write summaries)
- Only owner or shared user can call archive API

### Archive for Me Only (Owner)

- Sets `archivedByOwnerAt`, `archivedByOwnerUserId` on tree
- Does **not** set `isDeleted`; does **not** touch public summary
- Shared users keep full access; tree disappears from owner's main list, appears in "Archived by me"
- **Location**: `archiveTreeForOwnerOnly()`, `useArchiveTreeForOwnerOnly`

### Restore from Owner Archive

- Clears `archivedByOwnerAt`, `archivedByOwnerUserId`
- **Location**: `unarchiveTreeForOwner()`, `useUnarchiveTreeForOwner`

### Transfer Ownership

**API**: `POST /api/trees/transfer-ownership` — `treeId`, `newOwnerUserId`

**Rules**:
- Caller must be current tree owner
- New owner must be in `shared_data.sharedWith`
- Transaction: update tree `organizationId`, `createdBy`, `databaseCode`; adjust `tree_permissions` / `user_tree_permissions` (new owner as owner scope; previous owner typically gets editor permission)

**Location**: `app/api/trees/transfer-ownership/route.ts`, `ArchiveSharedTreeDialog` with transfer step

---

## Data Flow Diagrams

### Map Marker Visibility

```
useTrees(orgId)                    usePublicTreeSummaries()
       │                                    │
       ├─ listTrees(orgId)                  ├─ listTreePublicSummaries()
       └─ getTreesSharedWithUser(uid)       │
              │                             │
              ▼                             ▼
       trees (own + shared)           publicSummaries (all)
              │                             │
              │  ownTreeIds = Set(ids)      │  unrestrictedTrees = summaries
              │                             │    .filter(s => !ownTreeIds.has(s.id))
              ▼                             ▼
       TreeMarker (click → view-tree)  RestrictedTreeMarker (click → restricted-tree)
              zoom >= MIN_ZOOM               zoom >= 14 (public) or 18 (private)
```

### Share Link Redeem

```
User visits /share/{token}
       │
       ├─ Not signed in → getSignInUrl(/share/{token}) → redirect
       │
       └─ Signed in
              │
              ▼
       POST /api/share/{token}
       Authorization: Bearer <idToken>
              │
              ├─ Verify token → uid
              ├─ Get tree_share_links/{token} → treeId
              ├─ Check revoked, expiresAt
              ├─ Get trees/{treeId} → exists, !isDeleted
              ├─ tree_permissions + user_tree_permissions for uid
              ├─ Append accessHistory to tree
              └─ Return { treeId }
              │
              ▼
       Client: router.replace(/?drawer=view-tree&treeId={treeId})
```

### Restricted → Full View (After Approval)

```
User in restricted-tree drawer
       │
       useTree(treeId)  ── Firestore rules: no access → getTree throws or returns null
       useTreePublicSummary(treeId)  ── Allowed (public summaries readable by all)
       │
       ├─ fullTree is null → show limited info + "Request Access"
       │
       └─ Owner approves → grant tree permission → requester can read trees/{treeId}
              │
              useTree refetches (or invalidate) → fullTree exists
              │
              ▼
       useEffect: if (fullTree && treeId) navigate("view-tree", { treeId })
```

---

## Key Code Locations

| Feature | File(s) |
|---------|---------|
| Firestore rules (trees, tree_permissions, share links, access requests, public projections) | `firebase/firestore.rules` |
| Tree CRUD, listTrees, getTree | `lib/firebase/firestore.ts` |
| tree_permissions: grant, getTreesSharedWithUser, getTreeIdsSharedWithUser | `lib/firebase/firestore.ts`, `lib/api/grant-tree-access.ts`, `app/api/trees/[treeId]/permissions/route.ts` |
| Tree share links: create, revoke, list | `lib/firebase/firestore.ts` |
| Access requests: create, get, update | `lib/firebase/firestore.ts` |
| Public summaries: upsert, get, list, delete | `lib/firebase/firestore.ts` |
| Archive for owner only, unarchive | `lib/firebase/firestore.ts` |
| Share link redeem API | `app/api/share/[token]/route.ts` |
| Archive API (owner or shared user) | `app/api/trees/archive/route.ts` |
| Transfer ownership API | `app/api/trees/transfer-ownership/route.ts` |
| Admin: rebuild public map projection after support restore | `POST /api/trees/[treeId]/sync-public-projection` (`lib/server/sync-tree-public-projection-admin.ts`) |
| Admin: remove orphaned public docs | `POST /api/trees/cleanup-orphaned-public` |
| useTrees (own + shared merged) | `hooks/useTrees.ts` |
| useTreeSharedWith, useSharedTreeIds | `hooks/useTrees.ts`, `hooks/useSharedTreeIds.ts` |
| usePublicTreeSummaries | `hooks/usePublicTreeSummaries.ts` |
| Map markers (own vs restricted) | `components/map/map-markers.tsx` |
| View tree drawer | `app/drawers/view-tree/index.tsx` |
| Restricted tree drawer | `app/drawers/restricted-tree.tsx` |
| Share token page client | `app/share/[token]/share-token-client.tsx` |
| Archive shared tree dialog | `components/trees/archive-shared-tree-dialog.tsx` |
| Access request approval (Contact Us) | `app/drawers/contact-us.tsx` |
| Role permissions (team CRUD) | `lib/permissions-utils.ts` |

---

## Operations (support / admin)

- **`tree_public` / `tree_public_summaries`** are readable by all authenticated users but are only **updated** when someone can write the underlying tree (or via Admin). If ownership was fixed in `trees` only, map markers may still show **stale** creator or location until projections are rebuilt.
- **Rebuild after restore**: global admins can call **`POST /api/trees/[treeId]/sync-public-projection`** (Bearer ID token, same origin/CORS pattern as `cleanup-orphaned-public`) to re-sync public docs from `trees/{treeId}`.
- **Orphan markers** (tree hard-deleted without clearing public docs): **`POST /api/trees/cleanup-orphaned-public`**.

---

## Summary

- **Visibility**: Own + shared trees → full view; public summaries → restricted view (map markers, zoom ≥ 14 for public, ≥ 18 for private).
- **Privacy**: Firestore rules enforce access; **`tree_permissions`** grants per-tree sharing (legacy `shared_data` is deprecated).
- **Sharing**: Share links (redeem via API), username grant, or access request → approve.
- **Archive**: Full archive (soft delete) or owner-only archive (owner hides, shared users keep access).
- **Transfer**: Owner can transfer to a user who already has shared access; previous owner typically remains as collaborator via `tree_permissions`.

---

## Planned Upgrade (Migration Plan)

A phased migration plan is in place to upgrade the tree data structure before scale (currently ~300 trees). See the [Tree Database Structure Upgrade Plan](../../.cursor/plans/tree_database_structure_upgrade_c1d33ebe.plan.md).

### Target Architecture

Separation of concerns into four layers:

| Layer | Collection | Purpose |
|-------|------------|---------|
| **Canonical** | `trees/{treeId}` | Biological/operational truth only |
| **Governance** | `tree_governance/{treeId}` | Visibility, public settings, moderation |
| **Permissions** | `tree_permissions/{treeId}/members/{userId}` | Per-user roles (owner, manager, editor, contributor, viewer) |
| **Reverse index** | `user_tree_permissions/{userId}/trees/{treeId}` | Fast "trees shared with me" queries |
| **Public projection** | `tree_public/{treeId}` | Map-ready, privacy-filtered view (replaces tree_public_summaries) |

### Key Changes

- **Visibility levels**: `private`, `public` (explicit per-tree)
- **Roles**: Replace flat `sharedWith` array with role-based `tree_permissions`
- **Permission writes**: Via `POST /api/trees/[treeId]/permissions` (API, not client) — prevents malicious role escalation
- **Public projection**: Derived from `trees` + `tree_governance`; triggers on both writes; includes H3 geo indexing
- **Dual-read/dual-write**: During migration, rules allow both `shared_data` and `tree_permissions`; API dual-writes until deprecation

### Migration Phases

1. **Phase 1 (Foundation)**: Migration script creates tree_governance, tree_permissions, user_tree_permissions, tree_public; deploy dual-read rules + permissions API
2. **Phase 2**: Deprecate shared_data after validation
3. **Phase 3**: Governance UI + visibility filtering; Cloud Function triggers on trees + tree_governance for projection rebuild
4. **Phase 4 (Deferred)**: Extract history to tree_events
5. **Phase 5 (Deferred)**: Extract media to tree_media

### Related Documentation

- [Visibility & Permission Model v2](../architecture/visibility-permission-model-v2.md) — design spec
- [Tree Database Structure Upgrade Plan](../../.cursor/plans/tree_database_structure_upgrade_c1d33ebe.plan.md) — implementation plan
