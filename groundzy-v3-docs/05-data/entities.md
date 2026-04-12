# Entities

Catalog of **core entities** inferred from `types/*.ts` and **`firebase/firestore.rules`**. “Where it lives” is the primary Firestore collection unless noted.

---

## Geography & inventory

| Entity | Purpose | Key fields (high level) | Code | Used by |
|--------|---------|---------------------------|------|---------|
| **Tree** | Map-anchored tree record; inventory + history holder | `organizationId`, `databaseCode?`, `gzTin`, `species`, `location`, `propertyId?`, `clientId?`, `zoneId?`, `history?`, `isDeleted` | `types/tree.ts`, `trees` collection | Map, drawers, CRM, workflow line items, work_items |
| **Zone** | Polygon area; aggregate species counts | `organizationId`, `coordinates`, `propertyId?`, `clientId?`, `speciesCounts?` | `types/zone.ts`, `zones` | Map, zone services, requests |
| **ZoneService** | Bulk/zone-level service record | `zoneId`, `organizationId`, `serviceType`, `status`, `metadata` | `types/zone-service.ts`, subcollection `zones/{id}/zone_services` | Zone UI, work_items sync |
| **Property** | Site / parcel for CRM | `organizationId`, `address`, `clientId?`, `zoneId?`, `boundaries?` | `types/property.ts`, `properties` | Clients, trees, workflow |

---

## CRM

| Entity | Purpose | Key fields | Code | Used by |
|--------|---------|------------|------|---------|
| **Client** | Customer / account | `organizationId`, `type`, names/emails/phones, `contacts?` | `types/client.ts`, `clients` | Properties, workflow docs, trees |

---

## Workflow (commercial)

| Entity | Purpose | Key fields | Code | Used by |
|--------|---------|------------|------|---------|
| **Request** | Lead / service inquiry | `organizationId`, `clientId`, `propertyId?`, `requestItems?`, `status`, `convertedToQuoteId?`, `workItemIds?` | `types/request.ts`, `requests` | Quotes, jobs, work_items mirror |
| **Quote** | Estimate | `organizationId`, `clientId`, `propertyId`, `requestId?`, `lineItems`, `status`, `convertedToJobId?`, `workItemIds?` | `types/quote.ts`, `quotes` | Jobs, work_items mirror |
| **Job** | Scheduled work | `organizationId`, `clientId`, `propertyId`, `requestId?`, `quoteId?`, `treeIds?`, `pricing`, `status`, `workItemIds?` | `types/job.ts`, `jobs` | Invoices, work_items mirror |
| **Invoice** | Billing | `organizationId`, `clientId`, `jobId`, `quoteId?`, `lineItems`, `status`, `workItemIds?` | `types/invoice.ts`, `invoices` | Payments, work_items mirror |
| **RecurringPlan** | Repeating care schedule | `organizationId`, `targetKind`, `targetIds`, `frequency`, `nextRunAt` | `types/recurring-plan.ts`, `recurring_plans` | Generates work_items |

---

## Work & activity (cross-cutting)

| Entity | Purpose | Key fields | Code | Used by |
|--------|---------|------------|------|---------|
| **WorkItem** | Index row for ops + workflow mirrors | `organizationId`, `kind` (incl. `workflow_*`), `stage`, `treeIds`, `workflow?`, `source?` | `types/work-item.ts`, `work_items` | Dashboard, Activity tab, zones, invoices |
| **TreeEvent** | Append-only tree event (Phase 4) | `treeId`, `eventType`, `data` (HistoryEntry), timestamps | `types/tree.ts`, `tree_events/.../events` | Hydrating `tree.history` |
| **HistoryEntry** (embedded) | Union of service / inspection / note / measurement / access records | Varies by variant; `metadata` | `types/tree.ts`, embedded in `tree.history` | Tree history UI |

---

## Identity, teams, billing

| Entity | Purpose | Key fields | Code | Used by |
|--------|---------|------------|------|---------|
| **User** (document) | Profile, prefs, subscription, org link | `organizationId?`, `subscription`, `preferences`, `databaseCode?`, etc. | `lib/firebase/firestore.ts` (helpers), `users` collection | Auth, tier, work_items prefs |
| **Team** | Team subscription & membership | `ownerId`, `members`, `memberRoles`, `subscriptionStatus`, `tier?`, **`workflowSettings`** (embedded) | `types/team.ts`, `teams`; settings read/write `lib/firebase/firestore.ts` (`getTeamWorkflowSettings`) | Org-scoped data, quote/job defaults |
| **Organization** | Org document (rules reference `organizations/{orgId}`) | (see Firestore usage) | Referenced from `property`, `zone`, `types` comments | Scoping |
| **Username** | Unique handle mapping | `userId` | `usernames` | Profile, display |
| **InviteCode** | Team join | | `invite_codes` | Signup |
| **Session** | Device/session analytics | `userId`, `deviceType`, `startedAt` | `types/session.ts`, `users/.../sessions` | Analytics |

*Subscription fields* live on **user** and/or **team** documents; Stripe IDs in subscription blobs—see `lib/utils/tier-utils.ts`.

---

## AI & messaging

| Entity | Purpose | Key fields | Code | Used by |
|--------|---------|------------|------|---------|
| **AiChat** | Persisted AI chat thread | `userId`, `treeId?`, `title?` | `types/ai-chat.ts`, `ai_chats` | AI Chat drawer |
| **AiChatMessage** | Message in subcollection | `role`, `content`, attachments | `ai_chats/.../messages` | Chat UI |
| **Conversation** | Support or direct messaging | `type`, `participantIds` | `types/conversation.ts`, `conversations` | Inbox / contact |
| **Message** | Thread message | `senderId`, `body`, `attachments?` | `conversations/.../messages` | Messaging |

---

## Sharing, public, community (selected)

| Entity | Purpose | Code |
|--------|---------|------|
| **TreeShareLink** | Opaque token for share URL | `types/tree.ts`, `tree_share_links` |
| **TreePublicSummary** | Redacted public tree summary | `tree_public_summaries` |
| **TreeAccessRequest** | Request access to private tree | `tree_access_requests` |
| **Species catalog** | Taxonomy for quick picks / UI | `species_catalog`, `quick_pick_*` |
| **Community posts / Groundzy posts** | Social/feed | `community_posts`, `groundzy_posts` |

---

## Duplicates, overlaps, and naming (CRITICAL)

| Issue | Evidence |
|-------|----------|
| **Job vs Work Item** | **Job** is a workflow document (`jobs`). **WorkItem** includes `workflow_job` **mirror** kind — same business moment, two storage patterns (`types/work-item.ts` comment: LEGACY_MIRROR). |
| **Tree history vs TreeEvent vs WorkItem** | Embedded `TreeHistory`, subcollection `TreeEvent`, and `work_items` rows from `treeHistoryEntryToWorkItemFields` — triple path for overlapping semantics. |
| **Request vs tree_access_requests** | Different “request” words: CRM **Request** vs **TreeAccessRequest** for permissions — same English word, different domains. |
| **Team vs Organization** | User carries `organizationId`; `teams` and `organizations` collections both appear in rules — clarify relationship in v3. |
| **Weather** | Not a Firestore entity; **no** `WeatherRecord` collection—API-only. |

---

## Related

- [`relationships.md`](./relationships.md)
- [`event-system.md`](./event-system.md)
- [`data-model-overview.md`](./data-model-overview.md)
