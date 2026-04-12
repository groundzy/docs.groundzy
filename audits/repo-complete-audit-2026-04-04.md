# Groundzy Repository Complete Audit

Date: 2026-04-04  
Repo: `C:/Groundzy/app.groundzy`  
Auditor: Cursor AI assistant (code + runtime checks)

## Scope and Method

This audit covers the full repository state at audit time, including:

- Source/config/docs/security-rule structure review
- Runtime health checks (`build`, `test`, lint output review)
- API/documentation alignment checks
- Change-risk assessment for current working tree

Commands and evidence used:

- `npm run test` (Vitest)
- `npm run build` (Next.js production build)
- Existing terminal `npm run lint` output (reused to avoid duplicate run)
- Repository/file inventory via `git ls-files` and targeted code searches

---

## Executive Summary

The repository is **functional and deployable** (build/test pass), with strong domain modeling in `lib/groundzy` and Firestore security coverage. The main gaps are **code quality debt in lint warnings**, **documentation drift for new workflow APIs**, and **operational complexity risk** from a very large Firestore rules file and a high-volume in-flight change set.

Current readiness: **Good for active development**, **Moderate risk for release without cleanup gates**.

---

## Repository Snapshot

- Tracked files: **2100**
- Pending working tree changes: **64 paths** (modified + untracked)
- Top-level distribution (tracked):
  - `public`: 1223 files
  - `lib`: 237 files
  - `app`: 208 files
  - `components`: 113 files
  - `docs`: 89 files
  - `hooks`: 57 files
- Test files discovered: **46** (`lib/**/*.test.ts`)

Implication: runtime/domain logic is well represented in tests, but UI/app-route integration testing remains comparatively light.

---

## Runtime and Quality Health

## 1) Build Status

- `npm run build`: **PASS**
- Next.js 16.1.6 compiled successfully and generated routes/pages.
- No build-time TypeScript blockers observed.

Assessment: release artifact generation is healthy.

## 2) Automated Tests

- `npm run test`: **PASS**
- **46 test files**, **152 tests**, all passing.
- Coverage focus appears strongest in:
  - `lib/groundzy/*` event/projection flow
  - `lib/workflow/*`
  - `lib/utils/*`

Assessment: core domain behavior has meaningful test protection.

## 3) Lint Posture

- ESLint run result: **0 errors, 201 warnings**
- Current config in `eslint.config.mjs` intentionally downgrades several React-hook-related rules to warning:
  - `react-hooks/set-state-in-effect`
  - `react-hooks/preserve-manual-memoization`
  - others

Assessment: lint is non-blocking by design, but warning volume is high enough to hide real regressions.

---

## Architecture and Codebase Findings

### High Priority

1. **Large in-flight change set increases integration risk**
   - 64 changed paths across API routes, workflow/domain types, drawer UI, Firestore rules/indexes, and docs.
   - This is a broad blast radius touching both data model and UI behavior.
   - Risk: regressions hidden by mixed concerns in one branch.

2. **Lint debt is accumulated and normalized**
   - 201 warnings with relaxed rule severities in `eslint.config.mjs`.
   - Risk: performance/behavioral issues (especially hook lifecycle) can be missed until runtime.

3. **Firestore rules complexity is very high**
   - `firebase/firestore.rules` is extensive and centralizes many domains (auth, trees, workflow, social, messaging, admin).
   - Repeated cross-document `get/exists` patterns and broad policy surface increase maintenance and audit complexity.
   - Risk: accidental authorization gaps during feature changes.

### Medium Priority

4. **API reference drift**
   - New workflow participant endpoints exist in code:
     - `app/api/workflow/participant-list/route.ts`
     - `app/api/workflow/participant-get/[entity]/[id]/route.ts`
   - They are not yet documented in `docs/reference/api-routes.md`.
   - Risk: internal consumers and future contributors use stale docs.

5. **Testing scope imbalance (strong domain, weaker app/API integration)**
   - Tests are mostly in `lib/**/*.test.ts`; minimal route-level or UI behavior tests are visible.
   - Risk: contract regressions in `app/api/*` and drawer interactions slip through.

6. **Dev-only API safety depends on `NODE_ENV`**
   - `app/api/dev/create-user/route.ts` and `app/api/dev/simulate-payment/route.ts` gate via `process.env.NODE_ENV !== 'development'`.
   - Risk: accidental exposure in nonstandard environments where env flags are misconfigured.

### Low Priority

7. **README has onboarding noise/outdated steps**
   - Root README still emphasizes generic scaffolding and repeated install/setup instructions.
   - Repo-specific conventions are stronger in `docs/*` than in top-level README.

8. **Asset footprint concentration**
   - `public/` dominates tracked files.
   - Risk: repository growth and slower review cycles if not curated.

---

## Security and Access Model Notes

Positive signals:

- Firestore rules include default deny fallback.
- Sensitive server-side collections (`groundzy_events`, idempotency, quote tokens) are blocked from direct client access.
- Stripe webhook verifies signature (`stripe.webhooks.constructEvent`) and forces `nodejs` runtime.

Watchlist:

- Rules file breadth makes incremental validation critical after each permission-related change.
- Consider adding explicit production guardrails for dev endpoints beyond `NODE_ENV` checks.

---

## Documentation and Operational Maturity

Strengths:

- Documentation system is broad and organized (`docs/README.md`, architecture/features/reference maps).
- Domain docs for evolving workflow architecture are present (including new planning docs).

Gaps:

- API reference is not fully synchronized with newly added workflow participant routes.
- Root README can better reflect current production architecture and contributor flow.

---

## Prioritized Remediation Plan

## Next 7 days

1. Add workflow participant routes to `docs/reference/api-routes.md`.
2. Introduce lint warning budget (example: fail CI if warnings increase above baseline).
3. Split current branch work into smaller, reviewable PR-sized slices by domain:
   - workflow backend
   - UI drawer updates
   - rules/index changes
   - docs

## Next 30 days

4. Add API contract tests for critical routes:
   - `/api/groundzy/events/append`
   - `/api/workflow/participant-list`
   - `/api/workflow/participant-get/[entity]/[id]`
   - key share/sync endpoints
5. Add UI integration tests for core drawer workflows.
6. Add a deploy-time safety check that blocks dev endpoints unless explicit allow flag is set.

## Next 60 days

7. Refactor `firebase/firestore.rules` into a modular generation strategy (or documented policy sections + test harness) to reduce regression risk.
8. Establish release gate checklist:
   - build pass
   - tests pass
   - lint warning delta non-increasing
   - docs updated for all added public/internal APIs

---

## Overall Grade

- **Stability**: A-
- **Architecture clarity**: B+
- **Security posture**: B+
- **Testing maturity**: B
- **Documentation freshness**: B
- **Release hygiene (current branch state)**: C+

Composite assessment: **B (solid base, needs quality-gate tightening before high-velocity scaling).**

---

## Appendix: Key Evidence

- Build: passed (`next build`, Next.js 16.1.6)
- Tests: passed (`46 files`, `152 tests`)
- Lint: passed with warnings (`201 warnings`, `0 errors`)
- Working tree: `64` changed paths at audit time
- New workflow participant API routes detected in code but absent from API docs

