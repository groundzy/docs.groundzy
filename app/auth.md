# Auth Routes

Auth-related pages under `app/(auth)/`.

## Routes

| Path | Purpose |
|------|---------|
| `(auth)/login/page.tsx` | Login page (redirects to auth app) |
| `(auth)/auth/callback/page.tsx` | OAuth callback handler |
| `(auth)/payment/success/page.tsx` | Stripe payment success redirect |

## Auth Flow

- **Sign-in**: Redirects to `NEXT_PUBLIC_AUTH_APP_URL` (auth.groundzy.com)
- **Callback**: Handles OAuth return, creates/updates user doc
- **Payment success**: Post-checkout redirect, updates subscription

## Related

- [lib/auth-redirect.ts](../../lib/auth-redirect.ts) – `getSignInUrl()`
- [lib/firebase/auth.ts](../../lib/firebase/auth.ts) – Firebase Auth
- [components/auth/](../../components/auth/) – Auth provider, form
