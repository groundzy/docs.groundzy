# Scripts

Build and utility scripts in `scripts/`.

## Scripts

| Script | Purpose |
|--------|---------|
| seed-quick-picks.ts | Seed quick pick regions, sets, picks (Firebase Admin) |
| verify-api-routes.mjs | Verify API routes respond (BASE_URL) |
| generate-favicon.mjs | Generate favicon |

## Usage

```bash
npx tsx scripts/seed-quick-picks.ts
node scripts/verify-api-routes.mjs
node scripts/generate-favicon.mjs
```

## Dependencies

- **seed-quick-picks**: Requires FIREBASE_ADMIN_* env vars
- **verify-api-routes**: BASE_URL or localhost:3000
