# Coding Standards (Groundzy v3)

Rules to **prevent long-term drift**, aligned with [`Groundzy v3/00-foundation/principles.md`](../00-foundation/principles.md) and [`Groundzy v3/02-design/component-standards.md`](../02-design/component-standards.md).

---

## 1. TypeScript

- **`strict`** mode enabled; **no** new `// @ts-ignore` without ticket reference and removal plan.
- **Avoid `any`** — use `unknown` + narrowing, generics, or explicit domain types from `types/` / domain modules.
- **Public APIs** (exported hooks, repositories, utils) must have **explicit** return types where inference is fragile.
- **Zod** for runtime validation at boundaries (forms, API bodies, env parsing)—**single schema** per payload shape, shared between client and server when feasible.

---

## 2. Duplication and reuse

| Rule | Rationale |
|------|-----------|
| **No duplicated business logic** | Same calculation, status transition, or mapping in two files → extract to a **domain** module or `lib/<domain>/` util. |
| **Shared utilities only** | Helpers in `lib/` (or domain `utils/`) must be **generic** or **domain-named**—not feature-specific copy-paste. |
| **One formatter per concept** | Dates, money, addresses: use **one** module per concept (legacy has had duplicate formatters in drawers—avoid in v3). |
| **DRY with judgment** | Abstraction only when the **second** use case appears; premature abstractions are also debt. |

---

## 3. UI code

- **Features do not import** `@/components/ui/*` or `@radix-ui/*` — **Groundzy UI** only ([`component-standards.md`](../02-design/component-standards.md) § Rules).
- **No** ad hoc `className` soup for the same semantic (e.g. card padding)—use **tokens / Gz components**.

---

## 4. Data access

- **Firestore** access goes through **repository** or **firebase** helpers in domain modules—not raw `getDoc` scattered in components (v3 target).
- **Mutations** invalidate **query keys** via a **central** key factory per domain—no magic strings duplicated across files.

---

## 5. React

- **Prefer** small components; files **>400 lines** in orchestration layers trigger **extract** before new features (per existing drawer discipline).
- **Server components** where Next.js allows and data is static—**client** for map, drawers, interactivity.
- **Hooks** named `use*`; one hook file per concern when size grows.

---

## 6. Errors and logging

- **User-facing errors:** i18n keys + toast pattern; no raw exception strings in production UI.
- **Logging:** structured `console.error` in dev; avoid logging PII; server routes use minimal, actionable logs.

---

## 7. Imports

- **Order:** external → aliased (`@/`) → relative; group with blank lines (eslint can enforce).
- **Barrel files** (`index.ts`) allowed for **domains** and **Groundzy UI**—avoid deep circular barrels.

---

## 8. Security

- **Secrets** only in env / server; never in client bundles.
- **Firestore rules** are authoritative for client access; server routes must **mirror** invariants for Admin paths.

---

## 9. i18n

- **No** user-visible English strings in feature TSX without `t()` (or equivalent)—[`lib/i18n/`](../) patterns.

---

## 10. PR checklist (minimum)

- [ ] Types pass `tsc` / IDE
- [ ] `npm run lint` clean (or only allowed warnings)
- [ ] Tests added/updated for logic changes
- [ ] New UI uses **Groundzy** components (when package exists)
- [ ] Firestore rules / indexes updated if schema or queries change

---

## Related

- [`naming-conventions.md`](./naming-conventions.md)
- [`testing-strategy.md`](./testing-strategy.md)
- [`Groundzy v3/rules.md`](../rules.md)
