# Firebase App Hosting

Firebase App Hosting configuration in `apphosting.yaml`.

## Config

- **Region**: us-central1
- **Instances**: min 0, max 10, concurrency 80
- **Resources**: 1 CPU, **1024 MiB memory** (`runConfig.memoryMiB` — raised during Chromium/PDF experimentation; tune if workloads change.)

## Related

- **[PDF generation on App Hosting retrospective](./pdf-generation-app-hosting-retrospective.md)** — Quote/invoice PDF + email incidents (Puppeteer, Sparticuz, PDFKit, standalone tracing, git 408).

## Environment Variables

See `apphosting.yaml` for full list. Key groups:

- **Firebase client**: NEXT_PUBLIC_FIREBASE_*
- **Stripe**: NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY, STRIPE_SECRET_KEY (secret), STRIPE_WEBHOOK_SECRET (secret)
- **Firebase Admin**: FIREBASE_ADMIN_* (secrets)
- **URLs**: NEXT_PUBLIC_APP_URL, NEXT_PUBLIC_AUTH_APP_URL
- **Optional**: MAPBOX, PLANTNET, GOOGLE_PLACES, GEMINI, WEATHERAPI, VISUALCROSSING, RESEND

## Secrets

Set via Firebase CLI:

```bash
npx firebase apphosting:secrets:set STRIPE_SECRET_KEY --project <project-id>
npx firebase apphosting:secrets:set STRIPE_WEBHOOK_SECRET --project <project-id>
npx firebase apphosting:secrets:set FIREBASE_ADMIN_CLIENT_EMAIL --project <project-id>
npx firebase apphosting:secrets:set FIREBASE_ADMIN_PRIVATE_KEY --project <project-id>
# ... etc.
```
