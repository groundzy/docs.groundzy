# ESLint warnings audit — Groundzy app

**Last refreshed:** 2026-05-15  
**Workspace:** `Groundzy/app`  
**Command:** `npm run lint` → `eslint . --max-warnings 9999`  
**Result:** **115 warnings**, **0 errors** (lint exits successfully because `--max-warnings` allows warnings).

Related config: `eslint.config.mjs` at the root of the **`app`** repository (sibling of this `docs` repo); overview in [`docs/config/eslint.md`](../config/eslint.md).

**Progress:** Initial snapshot ~185 warnings → 164 (P0 a11y / rules-of-hooks / hygiene) → 142 (P1 exhaustive-deps pass) → 121 → **115** after `treesLimit` stabilization, `quote-form` / `job-form` effect deps, and `SpeciesIdentifierWizard` hook order + exhaustive deps.

---

## What are lint warnings?

**ESLint** analyzes JavaScript/TypeScript (and JSX here) for patterns that are likely bugs, regressions, accessibility issues, or maintenance problems.

| Severity | Meaning in this repo |
|----------|----------------------|
| **Error** | Fails `npm run lint` unless fixed or suppressed. |
| **Warning** | Reported but **does not fail** the lint script because `eslint.config.mjs` sets a high `--max-warnings` ceiling and several React Compiler / hooks rules are explicitly downgraded to `warn` (see block `groundzy/relax-for-now`). |

So **warnings are real signals**, but the project currently treats most of them as **technical debt** you can ship with—until you tighten the rules or lower `--max-warnings`.

---

## Should we plan to fix them?

**Yes—selectively.** Warnings are not “optional style”; they cluster into:

1. **Correctness / runtime risk** (hooks called conditionally, dependency arrays wrong → stale state).
2. **Accessibility** (missing/incorrect ARIA and alt text) — **P0 subset addressed (0 warnings)** for listed rules.
3. **Performance & React Compiler alignment** (effects that set state, memoization the compiler cannot preserve).
4. **Hygiene** (unused `eslint-disable`, unescaped JSX text).
5. **Next.js conventions** (`<img>` vs `next/image`).

You do **not** need a big-bang cleanup. A practical approach is **triage by rule** (below), fix **high-risk rules first**, and optionally **ratchet** quality later (e.g. lower `--max-warnings` or promote rules from `warn` to `error` file-by-file).

---

## Which ones to fix ASAP?

### P0 — Cleared (0 remaining for these rules)

| Rule | Count | Status |
|------|------:|--------|
| `react-hooks/rules-of-hooks` | **0** | Addressed. |
| `jsx-a11y/role-has-required-aria-props` | **0** | Addressed. |
| `jsx-a11y/alt-text` | **0** | Addressed. |

### P1 — Correctness / stale closures

| Priority | Rule | Count | Why soon |
|----------|------|------:|----------|
| **P1** | `react-hooks/exhaustive-deps` | **18** | Stale closures and missed updates (`mapbox-map`, species autocomplete, etc. still pending). |
| **P1** | `react-hooks/refs` | **21** | Refs in effects/cleanups. |

### P2 — Performance & React Compiler migration

| Priority | Rule | Count | Notes |
|----------|------|------:|------|
| **P2** | `react-hooks/set-state-in-effect` | **45** | Sync `setState` in `useEffect` (largest bucket now). |
| **P2** | `react-hooks/preserve-manual-memoization` | **6** | Compiler vs manual `useCallback` / `useMemo`. |
| **P2** | `react-hooks/immutability` | **2** | Compiler. |
| **P2** | `react-hooks/static-components` | **1** | Component identity. |

### P3 — Planned / batch cleanup

| Priority | Rule | Count | Notes |
|----------|------|------:|------|
| **P3** | `@next/next/no-img-element` | **22** | Prefer `next/image` when practical. |
| **P3** | `react/no-unescaped-entities` | **0** | Cleared in this refresh (quotes / apostrophes in JSX). |

---

## Full inventory by rule (115 total)

| Rule | Warnings | Category |
|------|----------:|----------|
| `react-hooks/set-state-in-effect` | 45 | Effects & state sync |
| `@next/next/no-img-element` | 22 | Next.js `<Image>` |
| `react-hooks/refs` | 21 | Refs in render/effects |
| `react-hooks/exhaustive-deps` | 18 | Dependencies / stale closures |
| `react-hooks/preserve-manual-memoization` | 6 | Compiler + memoization |
| `react-hooks/immutability` | 2 | Compiler |
| `react-hooks/static-components` | 1 | Component identity |

---

## Hot spots (files with the most warnings)

Paths below are relative to `app/` (the Next.js app root).

| Warnings | Path |
|----------:|------|
| 9 | `components/mapbox-map.tsx` |
| 7 | `drawers/client-form.tsx` |
| 7 | `drawers/property-form.tsx` |
| 4 | `components/trees/species-autocomplete.tsx` |
| 3 | `components/settings/address-autocomplete.tsx` |
| 3 | `components/trees/add-tree-form.tsx` |
| 3 | `components/welcome/welcome-intro-root.tsx` |
| 3 | `components/wizard/SpeciesIdentifierStep.tsx` |
| 3 | `components/work-area/work-area-mobile.tsx` |
| 3 | `hooks/useConversations.ts` |

Many other files have **1–3** warnings each; a JSON export from `eslint -f json` is the source of truth for exact line numbers.

---

## ESLint policy in this repo

From `eslint.config.mjs`:

- **Base:** `eslint-config-next`.
- **Ignores:** `.next`, `out`, `build`, `node_modules`, `docs`, `firebase`, `public`, and `*.config.*` variants.
- **Relaxation:** Several `react-hooks/*` rules are set to **`warn`** so lint stays green during incremental cleanup.

`npm run lint` uses `--max-warnings 9999`, so **warnings do not fail CI** unless that number is reduced.

---

## How to reproduce this audit

```bash
cd Groundzy/app
npm run lint
npx eslint . --max-warnings 9999 -f json -o eslint-report.json
```

---

## Suggested next steps (project-level)

1. Continue reducing **`react-hooks/exhaustive-deps`** (remaining hotspots: `mapbox-map.tsx`, `species-autocomplete`, `address-autocomplete`, workflow drawers).
2. **`react-hooks/set-state-in-effect`** — largest bucket; tackle by surface (forms, species autocomplete, map, etc.) or accept until React docs patterns are adopted repo-wide.
3. **`@next/next/no-img-element`** — migrate to `next/image` where layout and remote patterns allow.

---

*Regenerated to match ESLint JSON / `npm run lint` on 2026-05-15 (post-refresh: **115** warnings, **0** errors).*
