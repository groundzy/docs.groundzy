# Project Structure (Current)

Current directory layout and purpose of each major folder and file.

## Root Layout

```
app.groundzy/
├── app/                    # Next.js App Router
├── components/             # React components
├── lib/                   # Utilities, Firebase, i18n, AI, etc.
├── hooks/                 # Custom React hooks
├── stores/                # Zustand stores
├── types/                 # TypeScript types
├── public/                # Static assets
├── firebase/              # Firestore rules, indexes, storage rules
├── scripts/               # Build/seed scripts
├── apphosting.yaml        # Firebase App Hosting config
├── firebase.json          # Firebase project config
├── next.config.ts         # Next.js config
├── package.json
├── tailwind.config.ts
└── postcss.config.mjs
```

## app/

| Path | Purpose |
|------|---------|
| `app/(auth)/` | Auth routes: login, payment success, auth callback |
| `app/drawers/` | Drawer screen components (dashboard, trees, clients, etc.) |
| `app/api/` | Next.js API route handlers |
| `app/actions/` | Server actions (auth, payment, team) |
| `app/share/[token]/` | Public share-by-token page |
| `app/layout.tsx` | Root layout |
| `app/page.tsx` | Home page (redirects to app) |

## components/

| Path | Purpose |
|------|---------|
| `components/ui/` | shadcn-style UI primitives (button, input, dialog, etc.) |
| `components/map/` | Mapbox map, markers, layers, controls, draw, measure |
| `components/trees/` | Tree forms, lists, measurements, species autocomplete |
| `components/drawer-layout/` | Drawer layout primitives (header, body, footer) |
| `components/navigation/` | Sidebar, bottom nav |
| `components/work-area/` | Desktop/mobile work area |
| `components/auth/` | Auth provider, auth form |
| `components/weather/` | Weather cards, conditions |
| `components/wizard/` | Species identifier wizard |
| `components/chat/` | AI chat message content |
| `components/settings/` | Address autocomplete, etc. |
| `components/layout/` | App layout |

## lib/

| Path | Purpose |
|------|---------|
| `lib/firebase/` | Firebase config, Firestore, auth, admin |
| `lib/i18n/` | i18n config, messages, navigation labels |
| `lib/ai-chat/` | AI chat types, context, suggestions |
| `lib/weather/` | Weather API, H3, timeline, recommendations |
| `lib/drawer-registry.ts` | Drawer registration and metadata |
| `lib/drawers.ts` | Drawer definitions (40+ drawers) |
| `lib/drawer-utils.ts` | URL param parsing, buildDrawerUrl |
| `lib/utils/` | tier-utils, mapbox-forward-geocode, mapbox-reverse-geocode |
| `lib/stripe-config.ts` | Stripe price IDs, config |
| `lib/email.ts` | Resend email helpers |
| `lib/plantnet-client.ts` | PlantNet species ID (client-side) |

## hooks/

| Path | Purpose |
|------|---------|
| `hooks/useTrees.ts` | Tree CRUD, queries |
| `hooks/useSpecies.ts` | Species catalog, autocomplete |
| `hooks/useQuickPicks.ts` | Quick picks by region |
| `hooks/useAuth.ts` | Auth state, user |
| `hooks/useClients.ts`, `useProperties.ts` | CRM entities |
| `hooks/useRequests.ts`, `useQuotes.ts`, `useJobs.ts`, `useInvoices.ts` | Workflow entities |
| `hooks/useZones.ts` | Zone CRUD |
| `hooks/useTeams.ts` | Team management |

## stores/

| Path | Purpose |
|------|---------|
| `stores/navigation-store.ts` | Sidebar, work area, drawer history (persisted) |
| `stores/map-store.ts` | Map center, zoom, bounds |
| `stores/selection-store.ts` | Selected tree, zone, property |
| `stores/draw-store.ts` | Draw mode, polygon state |
| `stores/measure-store.ts` | Measure tool state |
| `stores/tree-add-store.ts` | Add-tree form state |
| `stores/auth-store.ts` | Auth UI state |
| `stores/fast-mapping-store.ts` | Fast mapping mode |
| `stores/identifying-wand-store.ts` | AI identify wizard state |
| `stores/view-tree-header-store.ts` | View-tree header state |
| `stores/breakout-placement-store.ts` | Breakout placement state |

## types/

| Path | Purpose |
|------|---------|
| `types/tree.ts` | Tree, ServiceType, HealthStatus, etc. |
| `types/zone.ts` | Zone, zone geometry |
| `types/client.ts` | Client |
| `types/property.ts` | Property |
| `types/request.ts` | Request |
| `types/quote.ts` | Quote |
| `types/job.ts` | Job |
| `types/invoice.ts` | Invoice |
| `types/species.ts` | Species catalog |
| `types/ai-chat.ts` | AI chat types |
| `types/conversation.ts` | Support/DM conversations |
| `types/quick-picks.ts` | Quick pick types |
| `types/team.ts` | Team, member roles |
| `types/gallery.ts` | User gallery |
| `types/map.ts` | Map types |
| `types/mapbox-gl-draw.d.ts` | Mapbox Draw type augmentations |

## firebase/

| Path | Purpose |
|------|---------|
| `firebase/firestore.rules` | Firestore security rules |
| `firebase/firestore.indexes.json` | Firestore composite indexes |
| `firebase/storage.rules` | Storage security rules |

## scripts/

| Path | Purpose |
|------|---------|
| `scripts/seed-quick-picks.ts` | Seed quick pick regions/sets/picks |
| `scripts/verify-api-routes.mjs` | Verify API routes respond |
| `scripts/generate-favicon.mjs` | Generate favicon |
