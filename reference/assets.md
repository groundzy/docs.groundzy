# Static Assets

Static assets in `public/`.

## Structure

| Path | Purpose |
|------|---------|
| public/images/ | Images, markers |
| public/images/markers/ | Map markers |
| public/images/markers/trees/ | Species markers (h-*, d-*, v-* SVGs) |
| public/images/markers/home.svg | Home marker |
| public/images/markers/property.svg | Property marker |
| public/logos/ | Logos |
| public/logos/logo-full.svg | Full logo |

## Species Markers

- Format: `h-{health}-{risk}.svg`, `d-*`, `v-*`
- Lookup: `lib/services/species-icon-service.ts`
