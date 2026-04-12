# Groundzy Documentation

Documentation for the Groundzy tree-mapping web application. This structure covers 100% of the codebase.

## Master Map

**[STRUCTURE.md](./STRUCTURE.md)** – Maps every path in the codebase to its documentation.

---

## Documentation Index

### Overview

| Document | Description |
|----------|-------------|
| [PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md) | Project overview, tech stack, features, getting started |

### Deployment

| Document | Description |
|----------|-------------|
| [deployment/DEPLOY.md](./deployment/DEPLOY.md) | Setup, deployment, environment variables |
| [deployment/firebase-app-hosting.md](./deployment/firebase-app-hosting.md) | Firebase App Hosting config |

### Architecture

| Document | Description |
|----------|-------------|
| [architecture/project-structure-current.md](./architecture/project-structure-current.md) | Current project structure |
| [architecture/complete-architecture-documentation.md](./architecture/complete-architecture-documentation.md) | Full architecture, patterns, data flow |
| [architecture/work-item-system-architecture.md](./architecture/work-item-system-architecture.md) | WorkItem as system of record; CRM, zones, history, Firestore, migration |
| [architecture/groundzy-work-domain-master-plan.md](./architecture/groundzy-work-domain-master-plan.md) | Master plan: ServiceType + WorkItem spine, repo reality vs aspirational docs, phased roadmap |
| [architecture/home-plus-to-pro-teams-linking.md](./architecture/home-plus-to-pro-teams-linking.md) | Foundation: connecting Home/Plus users to Pro/Teams accounts (directory ↔ org, relationships, resolution) |
| [architecture/quote-external-delivery-and-homeowner-signup.md](./architecture/quote-external-delivery-and-homeowner-signup.md) | Planned: quotes to non-Groundzy clients — SMS/email/PDF/print, QR, tokens, homeowner signup + CRM link |

### Audits & inventories

| Document | Description |
|----------|-------------|
| [audits/README.md](./audits/README.md) | Index — drawer audits, repo/pricing audits, PR checklist |

### Handoffs

| Document | Description |
|----------|-------------|
| [handoffs/README.md](./handoffs/README.md) | Index — refactor and workflow handoffs |

### Operations

| Document | Description |
|----------|-------------|
| [operations/README.md](./operations/README.md) | Index — pricing rollout notes, subscription backfills |

### App

| Document | Description |
|----------|-------------|
| [app/README.md](./app/README.md) | App directory layout |
| [app/auth.md](./app/auth.md) | Auth routes |
| [app/actions.md](./app/actions.md) | Server actions |
| [app/dev.md](./app/dev.md) | Dev tools |
| [app/drawers/README.md](./app/drawers/README.md) | Drawer components index |

### Components

| Document | Description |
|----------|-------------|
| [components/README.md](./components/README.md) | Components overview |
| [components/ui/README.md](./components/ui/README.md) | UI primitives |
| [components/map/README.md](./components/map/README.md) | Map components |
| [components/trees/README.md](./components/trees/README.md) | Tree components |
| [components/drawer-layout/README.md](./components/drawer-layout/README.md) | Drawer layout |
| [components/navigation/README.md](./components/navigation/README.md) | Sidebar, bottom nav |
| [components/work-area/README.md](./components/work-area/README.md) | Work area |
| [components/auth/README.md](./components/auth/README.md) | Auth components |
| [components/settings/README.md](./components/settings/README.md) | Settings |
| [components/layout/README.md](./components/layout/README.md) | App layout |

### Lib

| Document | Description |
|----------|-------------|
| [lib/README.md](./lib/README.md) | Lib overview |
| [lib/firebase/README.md](./lib/firebase/README.md) | Firebase utilities |
| [lib/i18n/README.md](./lib/i18n/README.md) | i18n |
| [lib/ai-chat/README.md](./lib/ai-chat/README.md) | AI chat lib |
| [lib/weather/README.md](./lib/weather/README.md) | Weather lib |
| [lib/utils/README.md](./lib/utils/README.md) | Utils |
| [lib/services/README.md](./lib/services/README.md) | Services |

### Hooks, Stores, Types

| Document | Description |
|----------|-------------|
| [hooks/README.md](./hooks/README.md) | Hooks index |
| [stores/README.md](./stores/README.md) | Zustand stores |
| [types/README.md](./types/README.md) | TypeScript types |

### Config

| Document | Description |
|----------|-------------|
| [config/README.md](./config/README.md) | Config overview |
| [config/nextjs.md](./config/nextjs.md) | Next.js config |
| [config/tailwind.md](./config/tailwind.md) | Tailwind config |
| [config/eslint.md](./config/eslint.md) | ESLint |
| [config/typescript.md](./config/typescript.md) | TypeScript |

### Firebase & Scripts

| Document | Description |
|----------|-------------|
| [firebase/README.md](./firebase/README.md) | Firebase config |
| [scripts/README.md](./scripts/README.md) | Build/utility scripts |

### Features

| Document | Description |
|----------|-------------|
| [features/README.md](./features/README.md) | Features index |
| [features/dashboard.md](./features/dashboard.md) | Dashboard |
| [features/trees.md](./features/trees.md) | Trees |
| [features/weather.md](./features/weather.md) | Weather |
| [features/ai.md](./features/ai.md) | AI (Identify, Chat) |
| [features/crm.md](./features/crm.md) | CRM |
| [features/map-and-zones.md](./features/map-and-zones.md) | Map, zones |
| [features/share-and-teams.md](./features/share-and-teams.md) | Share, teams |
| [features/profile-and-settings.md](./features/profile-and-settings.md) | Profile, settings |
| [features/hire-pro.md](./features/hire-pro.md) | Hire a Pro |
| [features/search.md](./features/search.md) | Search |
| [features/drawer-system.md](./features/drawer-system.md) | Drawer system |

### Reference

| Document | Description |
|----------|-------------|
| [reference/README.md](./reference/README.md) | Reference index |
| [reference/api-routes.md](./reference/api-routes.md) | API routes |
| [reference/environment-variables.md](./reference/environment-variables.md) | Environment variables |
| [reference/firestore-collections.md](./reference/firestore-collections.md) | Firestore |
| [reference/firestore-indexes.md](./reference/firestore-indexes.md) | Firestore indexes |
| [reference/storage-rules.md](./reference/storage-rules.md) | Storage rules |
| [reference/types-and-stores.md](./reference/types-and-stores.md) | Types and stores |
| [reference/dependencies.md](./reference/dependencies.md) | Dependencies |
| [reference/stripe.md](./reference/stripe.md) | Stripe |
| [reference/stripe-webhooks.md](./reference/stripe-webhooks.md) | Stripe webhooks (ops / endpoint checklist) |
| [reference/pricing-reference.md](./reference/pricing-reference.md) | Cross-repo pricing and tier reference |
| [reference/assets.md](./reference/assets.md) | Static assets |
| [reference/service-type-model-spec.md](./reference/service-type-model-spec.md) | ServiceType catalog enums and `ServiceTypeDefinition` (taxonomy spec) |

---

## Quick Links

- **Getting Started**: [PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md#getting-started)
- **Deployment**: [deployment/DEPLOY.md](./deployment/DEPLOY.md)
- **Full Codebase Map**: [STRUCTURE.md](./STRUCTURE.md)
- **Adding a Drawer**: [features/drawer-system.md](./features/drawer-system.md#adding-a-new-drawer)
- **API Reference**: [reference/api-routes.md](./reference/api-routes.md)
