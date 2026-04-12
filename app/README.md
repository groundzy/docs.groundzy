# App Directory

Next.js App Router structure for Groundzy.

## Layout

| Path | Purpose |
|------|---------|
| `app/layout.tsx` | Root layout, providers |
| `app/page.tsx` | Home page (redirects to app) |
| `app/providers.tsx` | React Query, Auth, Stripe, i18n providers |
| `app/globals.css` | Global styles, Tailwind |

## Route Groups

- **(auth)** – Login, auth callback, payment success → [auth.md](./auth.md)
- **share** – Public share token page → [features/share-and-teams.md](../features/share-and-teams.md)

## Subdirectories

| Directory | Purpose | Doc |
|-----------|---------|-----|
| `app/actions/` | Server actions (auth, payment, team) | [actions.md](./actions.md) |
| `app/api/` | API route handlers | [reference/api-routes.md](../reference/api-routes.md) |
| `app/drawers/` | Drawer screen components | [drawers/README.md](./drawers/README.md) |
