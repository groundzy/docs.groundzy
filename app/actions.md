# Server Actions

Next.js server actions in `app/actions/`.

## Files

| File | Purpose |
|------|---------|
| `auth.ts` | User profile creation, auth-related mutations |
| `payment.ts` | Stripe checkout flows, subscription creation |
| `team.ts` | Team creation, invite validation, member management |

## Usage

Server actions are called from client components via `"use server"` and handle server-side logic (Firebase Admin, Stripe, etc.) that cannot run in the browser.
