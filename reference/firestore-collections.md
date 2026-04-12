# Firestore Collections Reference

Firestore is the main database. Security rules are in `firebase/firestore.rules`. Indexes are in `firebase/firestore.indexes.json`.

## Access Patterns

- **Individual users**: `databaseCode` (from user doc) scopes trees, clients, properties, etc.
- **Teams**: `organizationId` scopes team data; `members` map for membership
- **Home-tier**: `organizationId == request.auth.uid` (legacy) or `userHasNoOrganization()`
- **Global admins**: `admins/{uid}` allowlist – read/write all where allowed

## Core Collections

### users/{userId}

User profile. Fields: `displayName`, `email`, `organizationId`, `databaseCode`, `role`, `subscription` (tier, status), `locale`, **`homePropertyId`** (optional; Home tier canonical personal property created from the first tree location), etc.

- **Manual QA (properties)**: Home — add tree without choosing a property → `trees.propertyId` and `users.homePropertyId` set; Plus — Property field and bulk “link to property” work; Pro — unchanged (client + property). See `components/trees/add-tree-form.tsx` and `app/drawers/trees.tsx`.

- **Subcollections**:
  - `users/{userId}/sessions/{sessionId}` – user sessions (UUID sessionId)
  - `users/{userId}/gallery/{photoId}` – My Photos
  - `users/{userId}/profile_public/profile` – public profile (displayName, photoURL)

### usernames/{username}

Username → userId mapping. Used for signup availability and profile URLs.

### teams/{teamId}

Team/organization. Fields: `ownerId`, `members`, `memberRoles`, etc.

### team_members/{memberId}

Team membership records. Fields: `teamId`, etc.

### organizations/{orgId}

Deprecated; kept for backward compatibility.

### invite_codes/{inviteCodeId}

Invite codes for team signup. Fields: `teamId`, etc.

## Tree & Map Data

### trees/{treeId}

Tree records. Fields: `databaseCode`, `organizationId`, `lat`, `lng`, `commonName`, `speciesId`, `height`, `dbh`, `healthStatus`, `riskLevel`, `isDeleted`, `createdAt`, etc.

- **Access**: organizationId, databaseCode, shared_data, or global admin

### zones/{zoneId}

Drawn polygons. Fields: `organizationId`, `geometry` (GeoJSON), `propertyId`, `isDeleted`, `createdAt`, etc.

- **Subcollection**: `zones/{zoneId}/zone_services/{serviceId}` – bulk zone services

### tree_number_counters/{scopeId}

Counters for tree numbers. `scopeId` = `databaseCode` (home) or `organizationId` (team).

### workflow_sequence_counters/{organizationId}

Per-team monotonic sequence fields for CRM workflow: `requestNext`, `quoteNext`, `jobNext`, `invoiceNext` (each is the next 1-based number to assign, same semantics as `tree_number_counters.nextNumber`). Used by Groundzy append transactions and client legacy creates.

### shared_data/{treeId}

Tree sharing. Document ID = treeId. Fields: `sharedWith` (array of userIds).

### tree_permissions/{treeId}/members/{userId}

Per-tree collaborator permission for a user. Typical fields: `role` (e.g. `editor`), `grantedByUserId`, `grantedAt`, `status`, optional **`grantSource`** (`user` \| `organization`), optional **`sourceOrganizationId`** when granted via org fan-out. Writes: Admin SDK / API routes only (rules block client creates).

### user_tree_permissions/{userId}/trees/{treeId}

Reverse index: trees shared with this user. Mirrors grants from `tree_permissions`. Writes: Admin SDK / API routes only.

### tree_share_links/{token}

Public share links. Document ID = token. Fields: `treeId`, `createdBy`, etc.

### tree_public_summaries/{summaryId}

Public tree summaries (summaryId = treeId). Limited fields for unrestricted markers.

### tree_access_requests/{requestId}

Access requests. Fields: `requesterId`, `ownerOrganizationId`, `status`, etc.

## CRM (Pro/Teams)

### clients/{clientId}

Client records. Fields: `organizationId`, `databaseCode`, name fields, `emails`, `phones`, `isDeleted`, optional **`homeownerUserId`** (Groundzy consumer uid after quote-portal claim, verified link, or **default-pro sync**), optional **`defaultProLinkedAt`** when created via `POST /api/user/sync-default-pro-share`. Deterministic id for that sync: `clients/gz_hm_{organizationId}_{homeownerUid}` (Admin SDK).

Soft-deleting a client unlinks optional `clientId` on **properties**, **trees**, and **zones** in the same org. Workflow documents (**requests**, **quotes**, **jobs**, **invoices**) keep `clientId` for audit and may store `clientDisplayNameSnapshot` when the client is removed so UIs can still show a stable name.

### properties/{propertyId}

Property records. Fields: `organizationId`, `clientId`, `address`, `geometry`, `isDeleted`, etc.

### requests/{requestId}

Request records. Fields: `organizationId`, `clientId`, optional `clientDisplayNameSnapshot`, `status`, optional `workItemIds` (links to `work_items` mirror rows), etc.

### quotes/{quoteId}

Quote records. Fields: `organizationId`, `clientId`, optional `clientDisplayNameSnapshot`, `requestId`, `status`, optional `workItemIds`, etc.

### jobs/{jobId}

Job records. Fields: `organizationId`, `clientId`, optional `clientDisplayNameSnapshot`, `status`, optional `workItemIds`, etc.

### invoices/{invoiceId}

Invoice records. Fields: `organizationId`, `clientId`, optional `clientDisplayNameSnapshot`, `jobId`, `status`, optional `workItemIds` (typically invoice mirror + job mirror), etc.

### recurring_plans/{planId}

Recurring maintenance templates (org-scoped). Fields include `organizationId`, `title`, `serviceTypeId`, `targetKind`, `targetIds`, `frequency`, `nextRunAt`, `isActive`. Client-side code creates scheduled `work_items` and advances `nextRunAt`. Composite index: `organizationId` + `isActive` + `targetIds` (array-contains) + `nextRunAt`.

### work_items/{workItemId}

Unified operational index: dual-written mirrors of tree history entries, zone services, and workflow documents (requests, quotes, jobs, invoices). Access rules align with jobs/requests (`organizationId`, optional `databaseCode`, home `organizationId == uid`). Deterministic doc ids: `th_{treeId}_{historyEntryId}`, `wf_{collection}_{docId}`, `zs_{zoneId}_{serviceId}`.

Optional facets (see `types/work-item.ts`): `serviceTypeId`, `assignment` (e.g. assignee user ids), `estimate` (rough totals from workflow pricing or service cost fields)—all optional and omitted when unknown.

**User preferences** (on `users/{uid}.preferences`): `workItemsDualWriteEnabled` (default on when omitted), `workItemsActivityTimelineEnabled`, `workItemsDashboardUpcomingEnabled` — see `lib/workflow/work-item-flags.ts`. When dashboard upcoming merge is on, the client loads future scheduled rows via [`listUpcomingScheduledWorkItemsForOrg`](../../lib/firebase/work-items.ts) (`stage == "scheduled"`, `schedule.start >= now`, ordered by `schedule.start` ascending).

**Backfill (Phase 5):** Existing data is populated as users edit trees or workflow docs (dual-write). A full historical backfill would re-save or run an admin script; do not remove legacy `trees.history` until parity is verified.

## Catalog & Config

### species_catalog/{speciesId}

Species catalog. Read: all authenticated. Write: global admins only.

### quick_pick_regions/{regionId}

Quick pick regions. Read: authenticated. Write: global admins.

### quick_pick_sets/{setId}

Quick pick sets. Fields: `regionId`, `isActive`, `version`. Read: authenticated. Write: global admins.

### quick_picks/{pickId}

Quick picks. Fields: `setId`, `isActive`, `sortOrder`. Read: authenticated. Write: global admins.

## Admin & System

### admins/{uid}

Global admin allowlist. Read: user can check own uid. Write: Admin SDK only.

### notifications/{notificationId}

System notifications (Stripe, **intelligence** alerts, etc.). **Create:** backend only (`firebase/firestore.rules`).

Typical fields: `userId`, `type`, `title`, `message`, `read`, `createdAt`. **Intelligence** rows may also include: `channel` (`app` \| `email` \| `sms`), `severity` (`info` \| `recommendation` \| `warning` \| `critical`), `sourceEventId` (links to `groundzy_events`), `organizationId`, `actions` (array of `{ kind, treeId?, route? }`), `dedupeKey`.

### groundzy_events/{eventId}

Append-only v3 event log (workflow, tree timeline, **intelligence**). **Client:** read/write denied; writes via Admin SDK / server routes only.

### groundzy_event_idempotency/{id}

Dedupe keys for event appends (user commands and **namespaced** `intel:…` keys for intelligence).

### mail/{mailId}

Trigger Email extension. Read: global admins. Write: none.

### pro_contact_requests/{requestId}

Hire Pro contact requests. Create: authenticated. Read: own or global admin. Optional `resolvedOrganizationId` when the contacted pro is a linked Groundzy Certified Pro (set by the app after `/api/resolve/pro-org`).

### groundzy_pros/{proId}

Groundzy Certified Pros. Read/write: global admins only (managed in `admin.groundzy`). Typical fields: `name`, `email`, `phone`, `website`, `formattedAddress`, `location`, `serviceRadiusMiles`, `status` (`active` \| `inactive`), optional `placeId` (Google Place ID for ratings/dedupe). **Linking:** optional `linkedOrganizationId` (team id or home-tier org scope used by CRM), optional `linkedAt`, `linkedByUid` when the link is set or changed.

### quote_portal_tokens/{token}

Opaque token for **public quote portal** (`/quote-portal/[token]`, `/api/quote-portal/*`). Fields: `quoteId`, `organizationId`, `clientId`, `status` (`active` \| `consumed` \| `revoked`), `expiresAt`, `createdAt`, `createdByUid`, optional `consumedAt`, `consumedByUid`. **Client SDK:** no access (`allow read, write: if false`); only Admin SDK / API routes.

## Conversations & AI

### conversations/{conversationId}

Support and direct messaging. Fields: `participantIds`, `type`, `lastMessageAt`, etc.

- **Subcollection**: `conversations/{id}/messages/{messageId}`

### ai_chats/{chatId}

AI Wizard chats. Fields: `userId`, etc.

- **Subcollection**: `ai_chats/{id}/messages/{messageId}`

## Social (Admin)

- `social_themes/{id}`
- `social_schedules/{id}`
- `social_posts/{id}`
- `social_assets/{id}`
- `social_settings/{id}`

All: global admins only.

## Composite Indexes

Defined in `firebase/firestore.indexes.json`. Key indexes:

- **trees**: `isDeleted` + `createdAt` (ASC/DESC)
- **zones**: `organizationId` + `isDeleted` + `createdAt`; `organizationId` + `isDeleted` + `propertyId` + `createdAt`
- **clients, properties**: `organizationId` + `isDeleted` + `createdAt`; properties also `clientId`
- **requests, quotes, jobs, invoices**: `organizationId` + `status`; `organizationId` + `clientId`; `organizationId` + `createdAt`; etc.
- **work_items**: `organizationId` + `isDeleted` + `updatedAt`; + `treeIds` (array-contains); + `zoneIds` (array-contains); + `clientId` / `propertyId` / `stage` with `updatedAt`; `organizationId` + `isDeleted` + `stage` + `schedule.start`
- **quick_picks**: `quick_pick_regions` (isDefault, name); `quick_pick_sets` (regionId, isActive, version); `quick_picks` (setId, isActive, sortOrder)
- **conversations**: `participantIds` (CONTAINS) + `lastMessageAt`; `type` + `lastMessageAt`
- **tree_access_requests**: `ownerOrganizationId` + `status`
- **notifications**: `userId` + `createdAt`
- **gallery** (collection group): `source` + `createdAt`

Deploy: `npx firebase deploy --only firestore:indexes`
