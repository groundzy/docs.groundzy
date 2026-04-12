# Naming Conventions (Groundzy v3)

## Files and folders

| Item | Convention | Example |
|------|------------|---------|
| **React components** | PascalCase | `TreeHistoryView.tsx` |
| **Hooks** | `use` + PascalCase file | `useTrees.ts` |
| **Utils / pure helpers** | kebab-case | `format-date.ts`, `tier-utils.ts` (match existing repo style where established) |
| **Types** | `types.ts` or `*.types.ts` inside domain | `types/tree.ts` legacy → domain `inventory/types.ts` v3 |
| **Tests** | Same base name + `.test.ts` | `tier-utils.test.ts` |
| **API routes** | Next App Router: `route.ts` in folder per path | `app/api/ai/chat/route.ts` |

**Domains:** use **lowercase** folder names (`inventory`, `crm`, `workflow`) for stability in imports.

---

## TypeScript symbols

| Symbol | Convention | Example |
|--------|------------|---------|
| **Components** | PascalCase | `GzButton`, `ViewTreeContent` |
| **Functions** | camelCase | `getEffectiveSubscriptionTier` |
| **Constants** | SCREAMING_SNAKE or `as const` objects | `PRICING_TIERS`, `WORKFLOW_DRAWERS` |
| **Interfaces / types** | PascalCase | `WorkItem`, `TreeHistory` |
| **Generics** | Single uppercase or descriptive | `TData`, `TEntity` |

---

## Groundzy UI

- **Prefix:** `Gz` + PascalCase — `GzButton`, `GzDrawerShell`, `GzFormField` ([`Groundzy v3/02-design/component-standards.md`](../02-design/component-standards.md)).
- **Primitive wrappers** may keep shadcn names **inside** `ui/` package only (not exported to features).

---

## React Query

- **Query keys:** **factory** per domain — e.g. `treeKeys.detail(treeId)`, `crmKeys.clients(orgId)` — **never** ad hoc string arrays in components.

---

## Firestore

- **Collection names:** existing **snake_case** or camelCase per repo—**do not** rename collections in v3 without migration; new collections follow **one** documented convention (prefer **camelCase** doc ids in code, collection names as existing rules).

---

## CSS / Tailwind

- **No** arbitrary class strings duplicated for the same semantic—use **components** or **@apply** in limited theme extensions (prefer components).

---

## Related

- [`coding-standards.md`](./coding-standards.md)
- [`file-structure.md`](./file-structure.md)
