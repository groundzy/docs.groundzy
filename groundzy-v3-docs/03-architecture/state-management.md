# State Management (Groundzy → v3)

## Current split (legacy)

Groundzy uses **two complementary** client state systems (`docs/architecture/complete-architecture-documentation.md`):

### Zustand — ephemeral UI and map state

| Store (examples) | Purpose |
|------------------|---------|
| `navigation-store` | Sidebar, work area, **drawer history** (back/forward), workflow dirty flags — **persisted** (partial) |
| `map-store` | Center, zoom, bounds |
| `selection-store` | Selected tree, zone, property |
| `draw-store`, `measure-store` | Tools |
| `auth-store` | Auth UI |
| Others | Tree add, identifying wand, view-tree header, etc. |

**Rule of thumb today:** If it is **not** persisted server-side and is about **UI or map interaction**, it often lives in Zustand.

### TanStack React Query — server-backed data

- **Trees, clients, properties, zones, requests, quotes, jobs, invoices**, work items, etc.
- Hooks in `hooks/` (e.g. `useTrees`, `useClients`) with query keys, invalidation, optimistic updates where implemented.

**Rule of thumb:** If it comes from **Firestore or API** and should **survive refresh** as shared truth, it goes through **Query**.

### Coupling

- Drawers **combine** both: Query for entity data, Zustand for navigation and dirty state.
- **URL** also holds drawer id and params — three sources of navigation truth (URL, persisted store, history stack) — documented in drawer navigation docs.

---

## v3 state management (target)

### Clear ownership

| State type | Owner | Notes |
|------------|-------|--------|
| **Server entities** | React Query (or successor) via **domain repositories** | Single query key convention per domain |
| **Map camera & tools** | Zustand or dedicated map store (platform) | Keep small |
| **Selection** | Platform store or URL-derived | Avoid duplicating selected id in three places |
| **Navigation** | Prefer **URL as source of truth** for drawer/route; minimize duplicate history in Zustand |
| **Form dirty / unsaved** | Single **navigation guard** abstraction (see `Groundzy v3/02-design/interaction-patterns.md`) |

### Anti-patterns to remove

- Storing **entity duplicates** in Zustand that should only live in Query cache.
- **Per-drawer** ad hoc loading flags when Query already exposes `isLoading` / `isFetching`.

### Data flow (v3)

```
User action
  → Domain hook (mutation)
  → Repository (Firestore / API)
  → Query invalidation
  → UI re-render

UI-only action (e.g. open panel)
  → Platform navigation adapter
  → URL + minimal client state
```

---

## Scalability

- **Query key factory** per domain (`['trees', treeId]`, `['clients', orgId]`) — documented and enforced.
- **Normalized caches** where relationships are heavy (optional optimization; product decision).

---

## Related

- [`routing-and-navigation.md`](./routing-and-navigation.md)
- [`frontend-architecture.md`](./frontend-architecture.md)
- `stores/navigation-store.ts` (legacy reference)
