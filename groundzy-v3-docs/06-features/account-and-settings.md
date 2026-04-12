# Account, profile & settings

## What it does

**Profile:** Account info, photo, subscription summary, Stripe portal, usage limits, locale, links to upgrades and **My Photos**.  
**Preferences:** Units, timezone, language, map marker size, **work_items** feature flags (`workItemsDualWriteEnabled`, Activity merge, dashboard merge)—see `UserPreferences` in `lib/firebase/firestore.ts`.  
**My Photos:** User gallery (`users/{userId}/gallery`).  
**Help / tutorial / contact-us:** FAQs, onboarding, **support & direct messaging** (`conversations` / `messages`).  
**Notifications:** `notifications` collection + dashboard badges.

## Who uses it

All tiers; **billing** actions depend on tier.

## Data involved

- **`users/{userId}`** — profile, subscription, preferences, `initialMapLocation`, etc.
- **Subcollections:** `sessions`, `gallery`, `profile_public/profile`
- **Storage:** avatars, gallery images
- **Messaging:** `conversations`, `messages` (`types/conversation.ts`)

## UI patterns

Multiple drawers (`profile`, `my-photos`, `help`, `tutorial`, `contact-us`)—**contact-us** is a large multi-mode “inbox” surface per drawer audits.

## Dependencies

- Firebase Auth
- Stripe portal API routes
- i18n locale
- Unread hooks for inbox badge

## Inconsistencies & overlaps

- **Preferences** control **data** behavior (dual-write, merged timelines)—crosses into **trees** and **dashboard** without living in those feature folders.
- **Profile** vs **public-profile** drawers—two profile concepts (self vs other user).
