# File Structure (Groundzy v3)

## Legacy structure (current app)

Per `docs/PROJECT_OVERVIEW.md`:

```
app/              # Next.js App Router: layouts, routes, api, drawers
components/       # UI, map, layout, domain-specific components
lib/              # Utilities, Firebase, i18n, AI, workflow, drawer helpers
hooks/            # React Query hooks and data hooks
stores/           # Zustand stores
types/            # Shared TypeScript types
firebase/         # firestore.rules, indexes, storage.rules
scripts/          # Migrations, seeds, verify scripts
public/           # Static assets
```

**Characteristics:** Flat `lib/` and large `app/drawers/` files; **domain** is implicit in folder names, not enforced by boundaries.

---

## Recommended v3 structure (target)

Aligns with [`Groundzy v3/03-architecture/system-overview.md`](../03-architecture/system-overview.md) and [`frontend-architecture.md`](../03-architecture/frontend-architecture.md).

```
app/                      # Routes: thin, compose domain + platform
  api/                    # Route handlers (unchanged conceptually)
  (auth)/                 # Auth routes
  share/                  # Public share
  ...

src/                      # Optional: if repo adopts src/ — otherwise top-level domains
  domains/
    inventory/            # Trees, zones, species, tree events
      types.ts
      repos/
      hooks/
      ui/
    crm/                  # Clients, properties
    workflow/             # Requests, quotes, jobs, invoices
    billing/              # Stripe integration surface (client)
    platform/             # Auth, navigation, map engine adapter
  ui/                     # Groundzy UI (design system) — wraps components/ui primitives

components/               # Transitional: shrink as domains/ui absorb
lib/                      # Transitional: shared cross-domain only (date, geo helpers)
```

**If the repo stays without `src/`:** use `domains/` at repo root or `lib/domains/`—**pick one** and document it in this file when adopted.

---

## Conventions

| Area | Convention |
|------|------------|
| **Domain** | Owns `types`, Firestore repos, hooks, feature UI under `domains/<name>/ui/`. |
| **Platform** | Map shell, auth session, router adapter—**no** CRM business rules. |
| **Groundzy UI** | Only presentation; **no** Firestore imports. |
| **App routes** | Import from `domains/*` and `platform/*`; **no** business logic in `page.tsx` beyond wiring. |
| **Tests** | Colocated `*.test.ts` / `*.test.tsx` next to unit under test, or `__tests__/` per domain—**one** convention per repo (choose and document). |

---

## Files to avoid

- **God files** — multi-thousand-line forms (legacy pattern); split into sections and hooks.
- **Misc `utils.ts`** at repo root — prefer **domain-scoped** or **well-named** `lib/format-date.ts` style.

---

## Related

- [`naming-conventions.md`](./naming-conventions.md)
- [`coding-standards.md`](./coding-standards.md)
