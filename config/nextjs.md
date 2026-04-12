# Next.js Configuration

`next.config.ts` configuration.

## Key Settings

| Setting | Value | Purpose |
|---------|-------|---------|
| allowedDevOrigins | ["http://192.168.1.168:3000"] | Dev CORS for network access |
| images.remotePatterns | firebasestorage.googleapis.com | Firebase Storage images |
| serverExternalPackages | firebase-admin, @google-cloud/firestore, @opentelemetry/api | Server externals |

## Scripts

- `npm run dev` – Next dev (binds 0.0.0.0)
- `npm run build` – Production build
- `npm run start` – Production server
