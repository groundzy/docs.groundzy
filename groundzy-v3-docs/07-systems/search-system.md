# Search system (cross-feature)

**Nature:** **Discovery** across entities—today implemented as **client-side filtering** over loaded collections, not a dedicated search microservice.

**Product/feature summary:** [`../06-features/search.md`](../06-features/search.md). **Authoritative** for duplicated UX bullets: link here instead of copying tier matrices.

---

## What exists

| Piece | Role |
|-------|------|
| **Global search drawer** | `app/drawers/search.tsx` — debounced query, groups results |
| **Data sources** | `useTrees`, `useZones`, `useProperties`, `useClients` (full lists), `useSearchSpecies` for catalog |
| **Scope** | Trees, zones, properties, clients, species — field lists in `docs/features/search.md` |
| **Tier gating** | Clients/properties hidden without `useProOrTeamsAccess()` |

**No** Firestore full-text index or Algolia in the surveyed feature doc—**O(n)** over in-memory lists for those entities.

---

## Duplication

- **Species** search overlaps **Explore**, **tree-add** autocomplete, **Quick Picks** — same catalog, **multiple** search UIs.
- **Map pan/zoom** discovery duplicates **text search** for “find my tree”—two UX systems for spatial data.

---

## Fragmentation

- **Search** is **not** a shared package—drawer-specific; **no** `SearchService` abstraction.
- **Results** navigate by **convention** (open `view-tree`, etc.)—not a typed `SearchResult` union with router handlers everywhere.

---

## Missing abstraction layers

| Gap | Impact |
|-----|--------|
| **Unified search index** | Large orgs: loading all clients/trees for client filter is **not** scalable. |
| **Query language** | No shared debounce + tokenizer + ranking helpers across search vs filter bars. |
| **Backend search** | Optional future: Algolia / Firestore extension—**not** present in repo as default. |

---

## v3 foundation direction

- **`SearchQuery`** type + **`SearchProvider`** interface (client index today, server index later).
- **Single** species resolution path for **search / explore / add tree**.
- **Pagination** or **server-side** query for CRM-scale lists before global search ships to big teams.

---

## Related

- `Groundzy v3/06-features/search.md`
- `docs/features/search.md`
