# Frontend Architecture (Groundzy → v3)

## Current architecture (legacy app)

### Framework and entry

- **Next.js 16+** with **App Router** (`app/`).
- Root layout wraps providers (React Query, i18n, theme, auth context).
- **Main app shell:** `components/layout/app-layout.tsx` — sidebar, **Mapbox** map, **work area** (desktop panel / mobile sheet).

### Composition model

- **Map-centric SPA:** Most product UI is not separate routes; it is **drawers** over the map (`docs/architecture/complete-architecture-documentation.md`).
- **Drawers** registered in `lib/drawers.ts` / `lib/drawer-registry.ts`, lazy-loaded, tier-gated via metadata.
- **Feature code** lives under `app/drawers/*`, `components/*` (map, trees, layout), with **direct** use of `components/ui` (shadcn) today.

### Map stack

- **Mapbox GL** + **Mapbox Draw** for polygons and tools; **Turf** / **h3-js** for geo operations.
- Map state in **Zustand** (`map-store`, `selection-store`, `draw-store`, `measure-store`, etc.).
- Tree/zone/property layers read from **React Query**–backed data.

### Component organization (today)

| Area | Location | Role |
|------|----------|------|
| Layout / shell | `components/layout/`, `components/work-area/` | App chrome, drawer host |
| Drawer primitives | `components/drawer-layout/` | DrawerShell, scroll, lists |
| UI primitives | `components/ui/` | shadcn/Radix |
| Domain UI | `components/trees/`, `components/mapbox-*`, etc. | Feature-specific |
| Drawers | `app/drawers/` | Screens and orchestration |

### Coupling issues

- **Drawers** mix **orchestration**, **data fetching**, and **raw UI primitives** in large files (see drawer audits in `docs/`).
- **No enforced domain boundary** — `lib/firebase` and hooks are imported from many drawers.
- **Design system** — Planned `Groundzy v3/02-design/` layer not present in legacy; features talk to shadcn directly.

---

## v3 frontend architecture (target)

### Layering

```
app/                    # Routes only — thin
  (routes import feature containers from domains/)

domains/<domain>/ui/   # Feature-specific UI using Groundzy UI only
platform/navigation/   # Shell, router adapter, drawer controller
ui/ (groundzy-ui)      # Design system — wraps shadcn
```

### Module boundaries

| Layer | Owns | Must not own |
|-------|------|--------------|
| **Route / drawer entry** | Params, layout, compose domain containers | Firestore field names, Stripe IDs |
| **Domain UI** | Screens for trees, CRM, workflow | Mapbox implementation details of other domains |
| **Groundzy UI** | Buttons, shells, forms, tables | Business rules |
| **Domain logic** | Validation, mapping, use-cases | React component state except UI-local |

### Data access from UI

- **Containers** call **hooks** exported from domains (e.g. `useTree`, `useUpdateClient`).
- Hooks internally use **React Query** + **repository** functions (Firestore or API).
- **No** Firestore imports inside presentational leaf components.

### Map in v3

- **Map engine** remains a thin **platform** module (Mapbox + layers).
- **Domains** subscribe to selection and submit geo updates via **typed contracts** (e.g. `MapSelectionPort`) to avoid circular imports between map and inventory.

---

## Scalability

- **Code-splitting:** Route and lazy drawer patterns continue; v3 adds **domain-level** lazy boundaries where useful.
- **Testing:** Domain use-cases testable without React; UI tests target Groundzy components and thin containers.

---

## Related

- [`system-overview.md`](./system-overview.md)
- [`state-management.md`](./state-management.md)
- [`routing-and-navigation.md`](./routing-and-navigation.md)
- [`Groundzy v3/02-design/design-system.md`](../02-design/design-system.md)
