# Components

React components in `components/`. Organized by domain and purpose.

## Directories

| Directory | Purpose | Doc |
|-----------|---------|-----|
| `ui/` | shadcn-style primitives | [ui/README.md](./ui/README.md) |
| `map/` | Mapbox map, markers, layers, controls | [map/README.md](./map/README.md) |
| `trees/` | Tree forms, lists, species, media | [trees/README.md](./trees/README.md) |
| `drawer-layout/` | Drawer header, body, footer, list | [drawer-layout/README.md](./drawer-layout/README.md) |
| `navigation/` | Sidebar, bottom nav | [navigation/README.md](./navigation/README.md) |
| `work-area/` | Desktop/mobile work area | [work-area/README.md](./work-area/README.md) |
| `auth/` | Auth provider, form | [auth/README.md](./auth/README.md) |
| `weather/` | WeatherCard, conditions | [features/weather.md](../features/weather.md) |
| `wizard/` | Species identifier wizard | [features/ai.md](../features/ai.md) |
| `chat/` | AI chat message content | [features/ai.md](../features/ai.md) |
| `settings/` | Address autocomplete | [settings/README.md](./settings/README.md) |
| `i18n/` | i18n provider | [lib/i18n/README.md](../lib/i18n/README.md) |
| `layout/` | App layout | [layout/README.md](./layout/README.md) |
| `bulk/` | Bulk share dialog | [features/share-and-teams.md](../features/share-and-teams.md) |
| `inbox/` | Inbox activity tab | [features/profile-and-settings.md](../features/profile-and-settings.md) |
| `stripe/` | Stripe provider | [reference/stripe.md](../reference/stripe.md) |
| `dev/` | Dev tools | [app/dev.md](../app/dev.md) |

## Root Components

| Component | Purpose |
|-----------|---------|
| `mapbox-map.tsx` | Main map container (also in map/) |
| `cookie-consent-banner.tsx` | Cookie consent |
| `upgrade-limit-cta.tsx` | Upgrade CTA for limits |
