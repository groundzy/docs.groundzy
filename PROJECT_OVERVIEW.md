# Groundzy – Project Overview

**Groundzy** is a tree-mapping web application for arborists, landscapers, and property managers. Users map trees on a Mapbox map, record species and measurements, manage clients and properties, and run workflows (requests, quotes, jobs, invoices). The app supports individual users (Home, Plus, Pro tiers) and teams (Small Team, Mid Team, Large Team, Enterprise).

## Tech Stack

| Category | Technology |
|----------|------------|
| **Framework** | Next.js 16+ (App Router) |
| **UI** | React 19, shadcn/ui (Radix primitives) |
| **Styling** | Tailwind CSS 4, tailwind-animate |
| **Client state** | Zustand |
| **Server/cache** | TanStack React Query |
| **Forms** | React Hook Form, Zod, @hookform/resolvers |
| **Maps** | Mapbox GL, @mapbox/mapbox-gl-draw |
| **Backend** | Firebase (Auth, Firestore, Storage) |
| **Payments** | Stripe |
| **AI** | Google Generative AI (Gemini) |
| **Icons** | Lucide React |
| **Other** | @turf/*, h3-js, date-fns, resend, browser-image-compression, html2canvas |

## Key Features

- **Map-centric UI**: Mapbox map with tree markers, zones (polygons), properties, drawing, and measurement tools
- **Drawer-based navigation**: 40+ drawers (dashboard, trees, clients, properties, jobs, invoices, AI chat, weather, etc.) opened via URL params `?drawer=<id>&<params>`
- **Trees**: CRUD, species catalog, quick picks, measurements, media, AI identification
- **CRM**: Clients, properties, requests, quotes, jobs, invoices (Pro/Teams tiers)
- **AI**: Groundzy Wizard, species identification, AI chat (Gemini)
- **Weather**: H3-based weather, timeline, recommendations
- **Share**: Public share links by token, tree access requests
- **Teams**: Invite codes, roles (owner, admin, manager, member)
- **i18n**: English, Spanish, French (via `lib/i18n/`, `useI18n()`, `t()`)

## Project Structure

```
app/           # Next.js App Router (pages, layouts, API routes, drawers)
components/    # React components (UI, map, auth, trees, etc.)
lib/           # Utilities, Firebase, i18n, AI, weather, drawer logic
hooks/         # Custom React hooks (data fetching, auth, trees, etc.)
stores/        # Zustand stores (navigation, map, selection, etc.)
types/         # TypeScript types
public/        # Static assets
firebase/      # Firestore rules, indexes, storage rules
scripts/       # Build/seed scripts
```

## Getting Started

1. **Install dependencies** (already run per workspace rules):
   ```bash
   npm install
   ```

2. **Environment variables**: Copy `.env.example` to `.env.local` and fill in values. See [reference/environment-variables.md](./reference/environment-variables.md).

3. **Run development server**:
   ```bash
   npm run dev
   ```

4. **Open** [http://localhost:3000](http://localhost:3000)

## Subscription Tiers

| Tier | Access |
|------|--------|
| Home | Basic trees, map, AI Wizard, weather |
| Plus | Home + Hire a Pro |
| Pro | Plus + Clients, Properties, full CRM |
| Small Team / Mid Team / Large Team / Enterprise | Pro + Requests, Quotes, Jobs, Invoices, Team Settings |

Tier logic: `lib/utils/tier-utils.ts`. Drawer visibility: `lib/drawers.ts`.

## Quick Picks (Add Tree species chips)

Quick Picks use Firestore collections `quick_pick_regions`, `quick_pick_sets`, and `quick_picks`. The API requires **composite indexes**.

1. **Seed data** (optional): `npx tsx scripts/seed-quick-picks.ts` (requires Firebase Admin env vars)
2. **Deploy indexes**:
   ```bash
   npx firebase use <your-project-id>
   npx firebase deploy --only firestore:indexes
   ```

Indexes are defined in `firebase/firestore.indexes.json`. Build can take a few minutes.

If you see **500** from `GET /api/quick-picks`, check dev server logs and browser console for the underlying error and Firestore index URL.
