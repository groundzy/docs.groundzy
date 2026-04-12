# Tech Stack (Groundzy v3)

## Current stack (legacy app — `package.json`)

| Layer | Technology | Notes |
|-------|------------|--------|
| **Runtime** | Node.js (via Next) | |
| **Framework** | Next.js ^16.x (App Router) | `app/` directory |
| **UI** | React ^19 | `react` / `react-dom` |
| **Language** | TypeScript ^5.7 | Strict mode expected |
| **Styling** | Tailwind CSS ^4, tailwind-animate | PostCSS pipeline |
| **Components** | shadcn/ui patterns (Radix primitives) | `components/ui/` |
| **Client state** | Zustand ^5 | `stores/` |
| **Server/cache** | TanStack React Query ^5 | `hooks/` |
| **Forms** | React Hook Form, Zod, @hookform/resolvers | |
| **Maps** | mapbox-gl, @mapbox/mapbox-gl-draw | Turf ^7, h3-js |
| **Backend (BaaS)** | Firebase ^12 (client), firebase-admin ^13 | Auth, Firestore, Storage |
| **Payments** | Stripe ^20, @stripe/react-stripe-js, @stripe/stripe-js | |
| **AI** | @google/generative-ai | Gemini |
| **Email** | resend | |
| **Icons** | lucide-react | |
| **Markdown** | react-markdown, remark-gfm | |
| **i18n** | App-owned (`lib/i18n/`) | en / es / fr |
| **Lint** | ESLint ^9, eslint-config-next ^16 | `eslint.config.mjs` |
| **Tests** | Vitest ^3 | `npm test` |

**Observability:** `@opentelemetry/api` present for optional instrumentation.

---

## Recommended stack for v3 (incremental)

**Retain** the same core: Next.js, React, TypeScript, Tailwind, Firebase, Stripe, TanStack Query, Zustand (scoped), RHF + Zod, Mapbox stack.

**Add / formalize:**

| Addition | Purpose |
|----------|---------|
| **Groundzy UI package** | Mandatory product-facing component layer over shadcn/Radix ([`Groundzy v3/02-design/design-system.md`](../02-design/design-system.md)) |
| **Domain modules** | Clear boundaries for inventory, CRM, workflow, map platform, billing ([`Groundzy v3/03-architecture/system-overview.md`](../03-architecture/system-overview.md)) |
| **ESLint import rules** | Block `@/components/ui` from feature folders once Groundzy UI exists |
| **Stricter CI** | Lint with **zero** warnings budget over time; typecheck + test on PR |

**Optional later:** Playwright or Cypress for E2E (not in current `package.json`); add when program funds it.

---

## Version policy

- **Pin major** framework versions in lockfile; upgrade Next/React/Firebase on a **scheduled** cadence with changelog review.
- **Security patches** applied promptly; feature upgrades require smoke test checklist.

---

## Related

- [`coding-standards.md`](./coding-standards.md)
- [`file-structure.md`](./file-structure.md)
