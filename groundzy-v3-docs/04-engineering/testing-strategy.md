# Testing Strategy (Groundzy v3)

## Current tooling

- **Runner:** Vitest ^3 (`npm test`, `vitest run` per `package.json`).
- **Lint:** ESLint with `eslint-config-next` — quality gate, not behavioral tests.

---

## Goals

| Goal | Approach |
|------|----------|
| **Prevent regressions** in business logic | Unit tests for pure functions: mappers, tier checks, formatters, Zod schemas |
| **Protect refactors** | Tests for hooks and repositories with mocked Firestore |
| **Avoid brittle UI** | Prefer **logic and integration** tests over snapshotting entire drawers |
| **Long-term drift** | New domain logic **ships with tests**; untested code is **exceptional** and flagged |

---

## What to test (priority)

### High priority

- **Domain logic:** `lib/workflow/*` adapters, `lib/utils/tier-utils.ts`, conversion helpers, money/date formatting.
- **Zod schemas** — valid/invalid cases.
- **Query key factories** — stable keys and invalidation lists.

### Medium priority

- **React hooks** — with `@testing-library/react` + Vitest (if/when added) or minimal render tests.
- **API route handlers** — mock `Request`, assert status and JSON shape (Stripe webhook signature tests in dedicated secure environment).

### Lower priority (until E2E infra exists)

- Full **browser** flows (map + drawer)—document manual QA checklist until Playwright/Cypress is adopted.

---

## What not to do

- **100% snapshot** of large components — high churn, low signal.
- **Testing implementation details** of React internals (`useState` call order).
- **Flaky** tests hitting real Firestore in CI — use **emulator** or **mocks**.

---

## CI expectations (v3 target)

| Stage | Command |
|-------|---------|
| **PR** | `npm run lint` + `npm test` + `tsc --noEmit` (or `next build` typecheck) |
| **Pre-deploy** | Same + `verify:api` / `verify:drawers` where applicable |

**Coverage:** Introduce a **minimum** threshold per domain over time (e.g. 60% lines on `domains/workflow`)—exact number is a program decision.

---

## Firebase / Emulator

- **Firestore rules** — use Firebase emulator suite or **rules unit tests** (`@firebase/rules-unit-testing`) for critical rules when feasible.
- **Document** emulator setup in `docs/` for contributors.

---

## Related

- [`coding-standards.md`](./coding-standards.md)
- [`tech-stack.md`](./tech-stack.md)
