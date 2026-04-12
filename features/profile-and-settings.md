# Profile and Settings Features

## Profile (`profile` drawer)

- **Location**: `app/drawers/profile.tsx`
- **Tier**: Home, Plus, Pro, Teams

### Sections

- **Account info**: Display name, email, photo
- **Subscription**: Current tier, Stripe portal link (manage payment, cancel)
- **Limits**: Tree usage, AI usage (from `useTreeUsage()`, `useAiUsage()`)
- **Upgrade**: Plus checkout (Home), Pro checkout (Plus)
- **Locale**: Language preference (en, es, fr) – stored in user doc
- **My Photos**: Link to `my-photos` drawer
- **Dev**: Development-only payment simulation button

### Data

- **User doc**: Firestore `users/{userId}` – displayName, email, photoURL, subscription, locale, databaseCode, organizationId, role
- **Avatar**: Firebase Storage `users/{userId}/profile/avatar`

## My Photos (`my-photos` drawer)

- **Location**: `app/drawers/my-photos.tsx`
- **Collection**: Firestore `users/{userId}/gallery`
- **Storage**: Firebase Storage `users/{userId}/gallery/{photoId}`
- **Index**: Collection group on `source` + `createdAt`
- **Usage**: User's photo library; can attach to AI Chat, etc.

## User Preferences

- **Hook**: `useUserPreferences()`
- **Storage**: Firestore `users/{userId}` or local
- **Fields**:
  - `isImperial`: true = F, in, ft; false = C, mm, m
  - `timezone`: For weather, dates
  - `locale`: en, es, fr (i18n)

## Help (`help` drawer)

- **Location**: `app/drawers/help.tsx`
- FAQs, support links

## Tutorial (`tutorial` drawer)

- **Location**: `app/drawers/tutorial.tsx`
- Tutorial content, links

## Contact Us / Inbox (`contact-us` drawer)

- **Location**: `app/drawers/contact-us.tsx`
- **Collection**: Firestore `conversations` (type: support), `conversations/{id}/messages`
- **Flow**: User creates support conversation; messages with Groundzy support
- **Unread**: `useUnreadMessages()` for badge

## Stripe Integration

- **Checkout**: `usePlusCheckout()`, `useProCheckout()` – create checkout session, redirect to Stripe
- **Portal**: `create-portal-session` – manage subscription, payment method
- **Webhook**: `app/api/stripe/webhook` – subscription updates, payment events
- **Config**: `lib/stripe-config.ts` – price IDs
