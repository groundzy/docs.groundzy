# Firebase

## What it does

**Backend-as-a-service:** Authentication, Firestore database, file Storage, optional Analytics; **Firebase Admin** on the server for privileged operations (custom tokens, webhooks, share resolution, batch jobs).

## How it’s used

### Client SDK (`firebase` package)

| Product | Use |
|---------|-----|
| **Auth** | User sessions (`lib/firebase/config.ts`) |
| **Firestore** | Primary app data (`lib/firebase/firestore.ts`, hooks) |
| **Storage** | Tree photos, user gallery, avatars |
| **Analytics** | Optional init when supported |

**Env:** `NEXT_PUBLIC_FIREBASE_*` — see `docs/reference/environment-variables.md`.

### Admin SDK (`firebase-admin`)

| Entry | Role |
|-------|------|
| **`lib/firebase/admin.ts`** | `adminAuth`, `adminDb` — singleton init with service account |
| **API routes / actions** | Stripe webhooks, share token, tree admin ops, AI persistence, team actions, etc. |

**Env:** `FIREBASE_ADMIN_PRIVATE_KEY`, `FIREBASE_ADMIN_CLIENT_EMAIL` (+ `NEXT_PUBLIC_FIREBASE_PROJECT_ID` for project id in cert).

### Rules & indexes

- **`firebase/firestore.rules`** — authoritative client access
- **`firebase/storage.rules`**
- **`firebase/firestore.indexes.json`** — composite indexes for queries

**Deploy:** `firebase deploy --only firestore|storage|hosting` (see `package.json` scripts).

## Risks & constraints

| Risk | Note |
|------|------|
| **Rules vs app logic** | Client tier gating must **not** replace rules; insecure rules = data leak |
| **Admin key custody** | Service account = full project power; server-only, secrets manager in prod |
| **Quota / costs** | Firestore reads/writes scale with usage; indexes required for complex queries |
| **Config validity** | `lib/firebase/config.ts` validates `NEXT_PUBLIC_*` before init |

## Related

- `Groundzy v3/05-data/*`
- `Groundzy v3/07-systems/permissions-system.md`
