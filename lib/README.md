# Lib Directory

Shared utilities, Firebase, i18n, and domain logic in `lib/`.

## Directories

| Directory | Purpose | Doc |
|-----------|---------|-----|
| firebase/ | Firebase config, Firestore, auth, admin | [firebase/README.md](./firebase/README.md) |
| i18n/ | i18n config, messages | [i18n/README.md](./i18n/README.md) |
| ai-chat/ | AI chat context, attachments, suggestions | [ai-chat/README.md](./ai-chat/README.md) |
| weather/ | Weather API, H3, recommendations | [weather/README.md](./weather/README.md) |
| utils/ | Utility functions | [utils/README.md](./utils/README.md) |
| tutorials/ | Tutorial definitions | [features/profile-and-settings.md](../features/profile-and-settings.md) |
| services/ | Services (species icon) | [services/README.md](./services/README.md) |

## Root Files

| File | Purpose |
|------|---------|
| drawer-registry.ts | Drawer registration |
| drawers.ts | Drawer definitions |
| drawer-utils.ts | URL param parsing |
| drawer-context.tsx | Drawer context |
| drawer-cache.ts | Drawer cache |
| drawer-spacing.ts | Drawer spacing |
| stripe-config.ts | Stripe price IDs |
| stripe-errors.ts | Stripe error handling |
| stripe-webhook-handlers.ts | Webhook handlers |
| auth-redirect.ts | Sign-in URL |
| auth-wait.ts | Auth wait |
| breakpoints.ts | Breakpoint helpers |
| card-styles.ts | Card CSS classes |
| consent.ts | Cookie consent |
| design-tokens.ts | Design tokens |
| geolocation.ts | GPS helpers |
| image-upload.ts | Image upload |
| map-constants.ts | Map style cycle |
| plantnet-client.ts | PlantNet API |
| profile-copy.ts | Limit/upgrade copy |
| tier-utils.ts | Subscription tier utils |
| utils.ts | cn, etc. |
| validate-image.ts | Image validation |
| work-area-variant-context.tsx | Work area variant |
| email.ts | Resend email |
| invite-code-utils.ts | Invite code helpers |
| permissions-utils.ts | Permission checks |
| username-utils.ts | Username helpers |
| settings-initial-map.ts | Initial map location |
