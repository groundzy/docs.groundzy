# Documentation Structure – 100% Coverage

This document maps every area of `app.groundzy` to its documentation. Each path links to the corresponding doc or folder.

## Docs Folder Tree

```
docs/
├── README.md                    # Main index
├── STRUCTURE.md                 # This file – full codebase map
├── PROJECT_OVERVIEW.md
│
├── deployment/
│   ├── DEPLOY.md
│   └── firebase-app-hosting.md
│
├── audits/                    # Audits, inventories, drawer PR checklist — see audits/README.md
├── handoffs/                  # Initiative handoffs — see handoffs/README.md
├── operations/                # Ops runbooks — see operations/README.md
│
├── architecture/
│   ├── intelligence-implementation-plan.md  # Phased build: intelligence / alerts / notifications
│   ├── intelligence/            # Derived events & intelligence layer — see intelligence/FILE-STRUCTURE.md
│   │   ├── README.md
│   │   └── FILE-STRUCTURE.md   # Full md manifest (existing v3 docs + gaps)
│   ├── project-structure-current.md
│   ├── complete-architecture-documentation.md
│   └── visibility-permission-model-v2.md
│
├── app/
│   ├── README.md
│   ├── auth.md
│   ├── actions.md
│   ├── dev.md
│   └── drawers/
│       └── README.md
│
├── components/
│   ├── README.md
│   ├── ui/README.md
│   ├── map/README.md
│   ├── trees/README.md
│   ├── drawer-layout/README.md
│   ├── navigation/README.md
│   ├── work-area/README.md
│   ├── auth/README.md
│   ├── settings/README.md
│   └── layout/README.md
│
├── lib/
│   ├── README.md
│   ├── firebase/README.md
│   ├── i18n/README.md
│   ├── ai-chat/README.md
│   ├── weather/README.md
│   ├── utils/README.md
│   └── services/README.md
│
├── hooks/README.md
├── stores/README.md
├── types/README.md
│
├── config/
│   ├── README.md
│   ├── nextjs.md
│   ├── tailwind.md
│   ├── eslint.md
│   └── typescript.md
│
├── firebase/README.md
├── scripts/README.md
│
├── features/
│   ├── README.md
│   ├── dashboard.md
│   ├── trees.md
│   ├── weather.md
│   ├── intelligence-alerts.md
│   ├── notification-center.md
│   ├── ai.md
│   ├── crm.md
│   ├── map-and-zones.md
│   ├── share-and-teams.md
│   ├── profile-and-settings.md
│   ├── hire-pro.md
│   ├── search.md
│   ├── drawer-system.md
│   └── plant-health-care.md
│
└── reference/
    ├── README.md
    ├── api-routes.md
    ├── environment-variables.md
    ├── intelligence-event-types.md
    ├── notification-types.md
    ├── firestore-collections.md
    ├── firestore-indexes.md
    ├── storage-rules.md
    ├── types-and-stores.md
    ├── dependencies.md
    ├── stripe.md
    ├── stripe-webhooks.md
    ├── pricing-reference.md
    └── assets.md
```

## Root

| Path | Documentation |
|------|---------------|
| `README.md` | Project README |
| `package.json` | [reference/dependencies.md](./reference/dependencies.md) |
| `next.config.ts` | [config/nextjs.md](./config/nextjs.md) |
| `tailwind.config.ts` | [config/tailwind.md](./config/tailwind.md) |
| `apphosting.yaml` | [deployment/firebase-app-hosting.md](./deployment/firebase-app-hosting.md) |
| `firebase.json` | [firebase/README.md](./firebase/README.md) |

---

## app/

| Path | Documentation |
|------|---------------|
| `app/layout.tsx` | [app/README.md](./app/README.md#layout) |
| `app/page.tsx` | [app/README.md](./app/README.md#pages) |
| `app/providers.tsx` | [app/README.md](./app/README.md#providers) |
| `app/globals.css` | [config/tailwind.md](./config/tailwind.md) |
| `app/(auth)/` | [app/auth.md](./app/auth.md) |
| `app/share/` | [features/share-and-teams.md](./features/share-and-teams.md) |
| `app/actions/` | [app/actions.md](./app/actions.md) |
| `app/api/` | [reference/api-routes.md](./reference/api-routes.md) |
| `app/drawers/` | [features/drawer-system.md](./features/drawer-system.md), [app/drawers/README.md](./app/drawers/README.md) |

---

## components/

| Path | Documentation |
|------|---------------|
| `components/ui/` | [components/ui/README.md](./components/ui/README.md) |
| `components/map/` | [features/map-and-zones.md](./features/map-and-zones.md), [components/map/README.md](./components/map/README.md) |
| `components/trees/` | [features/trees.md](./features/trees.md), [components/trees/README.md](./components/trees/README.md) |
| `components/drawer-layout/` | [components/drawer-layout/README.md](./components/drawer-layout/README.md) |
| `components/navigation/` | [components/navigation/README.md](./components/navigation/README.md) |
| `components/work-area/` | [architecture/complete-architecture-documentation.md](./architecture/complete-architecture-documentation.md), [components/work-area/README.md](./components/work-area/README.md) |
| `components/auth/` | [app/auth.md](./app/auth.md), [components/auth/README.md](./components/auth/README.md) |
| `components/weather/` | [features/weather.md](./features/weather.md) |
| `components/wizard/` | [features/ai.md](./features/ai.md) |
| `components/chat/` | [features/ai.md](./features/ai.md) |
| `components/settings/` | [components/settings/README.md](./components/settings/README.md) |
| `components/i18n/` | [lib/i18n/README.md](./lib/i18n/README.md) |
| `components/layout/` | [components/layout/README.md](./components/layout/README.md) |
| `components/bulk/` | [features/share-and-teams.md](./features/share-and-teams.md) |
| `components/inbox/` | [features/profile-and-settings.md](./features/profile-and-settings.md) |
| `components/stripe/` | [reference/stripe.md](./reference/stripe.md) |
| `components/dev/` | [app/dev.md](./app/dev.md) |
| Root components | [components/README.md](./components/README.md) |

---

## lib/

| Path | Documentation |
|------|---------------|
| `lib/firebase/` | [lib/firebase/README.md](./lib/firebase/README.md) |
| `lib/i18n/` | [lib/i18n/README.md](./lib/i18n/README.md) |
| `lib/ai-chat/` | [features/ai.md](./features/ai.md), [lib/ai-chat/README.md](./lib/ai-chat/README.md) |
| `lib/weather/` | [features/weather.md](./features/weather.md), [lib/weather/README.md](./lib/weather/README.md) |
| `lib/utils/` | [lib/utils/README.md](./lib/utils/README.md) |
| `lib/tutorials/` | [features/profile-and-settings.md](./features/profile-and-settings.md) |
| `lib/services/` | [lib/services/README.md](./lib/services/README.md) |
| `lib/drawer-*.ts` | [features/drawer-system.md](./features/drawer-system.md) |
| `lib/stripe-*.ts` | [reference/stripe.md](./reference/stripe.md) |
| Other lib files | [lib/README.md](./lib/README.md) |

---

## hooks/

| Path | Documentation |
|------|---------------|
| All hooks | [hooks/README.md](./hooks/README.md) |

---

## stores/

| Path | Documentation |
|------|---------------|
| All stores | [reference/types-and-stores.md](./reference/types-and-stores.md), [stores/README.md](./stores/README.md) |

---

## types/

| Path | Documentation |
|------|---------------|
| All types | [reference/types-and-stores.md](./reference/types-and-stores.md), [types/README.md](./types/README.md) |

---

## firebase/

| Path | Documentation |
|------|---------------|
| `firebase/firestore.rules` | [reference/firestore-collections.md](./reference/firestore-collections.md) |
| `firebase/firestore.indexes.json` | [reference/firestore-indexes.md](./reference/firestore-indexes.md) |
| `firebase/storage.rules` | [reference/storage-rules.md](./reference/storage-rules.md) |
| `firebase/storage.cors.json` | [firebase/README.md](./firebase/README.md) |

---

## scripts/

| Path | Documentation |
|------|---------------|
| All scripts | [scripts/README.md](./scripts/README.md) |

---

## public/

| Path | Documentation |
|------|---------------|
| `public/images/` | [reference/assets.md](./reference/assets.md) |
| `public/logos/` | [reference/assets.md](./reference/assets.md) |

---

## Features (by drawer)

| Drawer | Documentation |
|--------|----------------|
| dashboard | [features/dashboard.md](./features/dashboard.md) |
| trees | [features/trees.md](./features/trees.md) |
| tree-add | [features/trees.md](./features/trees.md) |
| view-tree | [features/trees.md](./features/trees.md) |
| edit-tree | [features/trees.md](./features/trees.md) |
| weather | [features/weather.md](./features/weather.md) |
| ai-chat | [features/ai.md](./features/ai.md) |
| ai-identifying-wand | [features/ai.md](./features/ai.md) |
| clients, properties | [features/crm.md](./features/crm.md) |
| requests, quotes, jobs, invoices | [features/crm.md](./features/crm.md) |
| draw, measure | [features/map-and-zones.md](./features/map-and-zones.md) |
| share | [features/share-and-teams.md](./features/share-and-teams.md) |
| team-settings | [features/share-and-teams.md](./features/share-and-teams.md) |
| profile, my-photos | [features/profile-and-settings.md](./features/profile-and-settings.md) |
| hire-groundzy-pro | [features/hire-pro.md](./features/hire-pro.md) |
| search | [features/search.md](./features/search.md) |
| help, tutorial, contact-us | [features/profile-and-settings.md](./features/profile-and-settings.md) |
| more | [features/drawer-system.md](./features/drawer-system.md) |
