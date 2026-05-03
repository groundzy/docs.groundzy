# Relationships

How entities **connect** in the current model. Foreign keys are typically **string ids** on documents; Firestore is **not** a relational DB—integrity is enforced in **rules**, **app logic**, and **indexes**.

---

## 1. Diagram (workflow)

```
Client ←──┬── Property ←── Tree (optional many trees per property)
          │       ↑
          │       └── Zone (optional)
          │
          ├── Request ──→ Quote ──→ Job ──→ Invoice
          │       │          │         │
          │       └──────────┴─────────┴── (optional requestId / quoteId links)
          │
          └── (contactId on workflow docs → Client.contacts)
```

---

## 2. Core foreign keys

| From | To | Field(s) | Notes |
|------|-----|----------|--------|
| **Tree** | Property | `propertyId?` | Optional |
| Tree | Client | `clientId?` | Optional |
| Tree | Zone | `zoneId?`, `brokenOutFromZoneId?` | Assigned zone membership and lineage from aggregate |
| **Property** | Client | `clientId?` | Optional (Plus comment in `types/property.ts`) |
| **Zone** | Property / Client | `propertyId?`, `clientId?` | Optional |
| **Zone** | Aggregate trees | `speciesCounts[]` | Grouped-tree inventory without individual Tree docs |
| **Request** | Client, Property | `clientId`, `propertyId?` | Required client |
| **Quote** | Client, Property, Request | `clientId`, `propertyId`, `requestId?` | |
| **Job** | Client, Property, Request, Quote | `clientId`, `propertyId`, `requestId?`, `quoteId?` | |
| **Invoice** | Client, Job, Quote | `clientId`, `jobId`, `quoteId?` | `jobId` required on type |
| **RequestItem** | Tree or Zone | `treeId?` / `zoneId?` | Exactly one per item (`types/request.ts`) |

---

## 3. User & team

| From | To | Field | Notes |
|------|-----|-------|--------|
| **User** doc | Team / Org | `organizationId` | Scopes org data access |
| **Tree** | Org scope | `organizationId`, `databaseCode?` | Solo vs org |
| **Team** | Users | `members`, `memberRoles` | Parallel to `team_members` collection in rules |

---

## 4. Work items

| From | To | Field | Notes |
|------|-----|-------|--------|
| **WorkItem** | Trees / zones / property / client | `treeIds[]`, `zoneIds?`, `propertyId?`, `clientId?` | |
| WorkItem | Workflow docs | `workflow.requestId` etc. | Optional links |
| WorkItem | Tree history | `source.historyEntryId` | Dedupe in UI |
| CRM entities | WorkItem rows | `workItemIds?` on Request/Quote/Job/Invoice | Mirror linkage |

---

## 5. Sharing & permissions (tree)

| Mechanism | Purpose |
|-----------|---------|
| `tree_permissions/{treeId}/members/{userId}` | Per-tree ACL |
| `user_tree_permissions/{userId}/trees/{treeId}` | Inverse index |
| `tree_share_links/{token}` | Public link |
| Rules note: **shared_data deprecated** — use tree_permissions |

---

## 6. Gaps and indirect connections

| Gap | Evidence |
|-----|----------|
| **Conversion backlinks** | Historical audits: `convertedTo*` fields not always written; workflow upgrade docs describe fixes—verify current code paths. |
| **No DB-level FK enforcement** | Deletes may orphan references; soft-delete snapshots (`clientDisplayNameSnapshot`) compensate for CRM. |
| **Job ↔ Tree** | Primarily `treeIds` on Job + line items; deprecated `tree.business.activeJobIds` on Tree (`types/tree.ts`). |
| **Invoice ↔ Tree** | Via line item `treeIds`, not a single tree id on invoice header. |

---

## 7. Tree species and species catalog

| From | To | Field | Notes |
|------|-----|-------|-------|
| **Tree** | **species_catalog** | `species.catalogItemId?` | Optional link to curated catalog docs. Nested `species.*` blocks may be omitted when the catalog is authoritative and unchanged in the UI; clients merge catalog rows at read time for display (see `mergeTreeSpeciesWithCatalogItem` / `buildSpeciesPayloadForSave` under `Groundzy/app/lib/trees/`). |

---

## Related

- [`entities.md`](./entities.md)
- [`data-flows.md`](./data-flows.md)
- [`permissions.md`](./permissions.md)
