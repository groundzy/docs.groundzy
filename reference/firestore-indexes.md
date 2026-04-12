# Firestore Indexes

Composite indexes defined in `firebase/firestore.indexes.json`.

## Indexes by Collection

| Collection | Fields | Purpose |
|------------|--------|---------|
| gallery (collection group) | source, createdAt | Gallery queries |
| trees | isDeleted, createdAt (ASC/DESC) | Tree list |
| quick_pick_regions | isDefault, name | Default region lookup |
| quick_pick_sets | regionId, isActive, version | Sets by region |
| quick_picks | setId, isActive, sortOrder | Picks by set |
| zones | organizationId, isDeleted, createdAt | Zone list |
| zones | organizationId, isDeleted, propertyId, createdAt | Zones by property |
| conversations | participantIds (CONTAINS), lastMessageAt | User conversations |
| conversations | type, lastMessageAt | Conversations by type |
| tree_access_requests | ownerOrganizationId, status | Access requests |
| clients | organizationId, isDeleted, createdAt | Client list |
| properties | organizationId, isDeleted, createdAt | Property list |
| properties | organizationId, isDeleted, clientId, createdAt | Properties by client |
| requests | organizationId, status | Request list |
| requests | organizationId, clientId | Requests by client |
| requests | organizationId, createdAt | Requests by date |
| quotes | organizationId, status | Quote list |
| quotes | organizationId, clientId | Quotes by client |
| quotes | organizationId, requestId | Quotes by request |
| quotes | organizationId, createdAt | Quotes by date |
| jobs | organizationId, status | Job list |
| jobs | organizationId, clientId | Jobs by client |
| jobs | organizationId, createdAt | Jobs by date |
| invoices | organizationId, status | Invoice list |
| invoices | organizationId, clientId | Invoices by client |
| invoices | organizationId, jobId | Invoices by job |
| invoices | organizationId, createdAt | Invoices by date |
| notifications | userId, createdAt | User notifications |

## Deploy

```bash
npx firebase deploy --only firestore:indexes
```

Build can take several minutes.
