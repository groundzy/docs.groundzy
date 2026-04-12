# Intelligence notification preferences — implementation summary

**Status:** Implemented (app + auth).  
**Related:** [notification-channel-routing.md](./notification-channel-routing.md), [alert-notification-system.md](./alert-notification-system.md).

---

## Data model

Firestore **`users/{userId}.notificationPreferences`** (top-level, not under `preferences`):

```ts
notificationPreferences: {
  intelligence: {
    email: boolean; // intelligence email opt-in
    sms: boolean;   // user intent for SMS; effective delivery also requires phone + tier
  };
}
```

**Phone** remains **`users/{userId}.phone`** (single source for SMS delivery).

**Defaults (missing document or missing booleans)** are applied only in **`resolveIntelligenceNotificationPrefs`** (`app/lib/notifications/resolve-intelligence-notification-prefs.ts` in the app repo):

- **Email:** `true` (Option A — continuity with pre-prefs behavior).
- **SMS intent:** `false`.
- **Effective `smsCriticalOptIn`:** `sms === true` **and** non-empty `phone` (tier gating is separate, in `decideIntelligenceChannels`).

---

## Notification delivery rules (MUST)

- If `notificationPreferences.intelligence.email === false` → **do not send** intelligence email.
- If `notificationPreferences.intelligence.sms === false` → **do not send** intelligence SMS.
- **No inline defaults** in evaluators or senders — use **`resolveIntelligenceNotificationPrefs`** only.
- **Pipeline:** `resolveIntelligenceNotificationPrefs(userData)` → `decideIntelligenceChannels` → `sendIntelligenceEmailIfNeeded` / `sendIntelligenceSmsIfNeeded`.

Storm evaluation uses this path: `app/lib/intelligence/run-storm-weak-tree-evaluation.ts` (app repo).

---

## App (app.groundzy)

- **Firestore:** `lib/firebase/firestore.ts` — `updateUserNotificationPreferences` (client-side update).
- **Settings:** Profile drawer → **Intelligence alerts** collapsible — `app/drawers/profile/ProfileIntelligenceNotificationsCollapsible.tsx`. SMS toggle disabled without Teams-tier capability (`notifications.sms_critical`) or without a saved phone.
- **i18n:** `profile.intelligenceNotif.*` in `lib/i18n/messages.ts`.

---

## Auth (auth.groundzy)

- **Firestore:** `lib/firebase/firestore.ts` — `updateUserNotificationPreferences`.
- **Onboarding:** Units step — checkboxes for email / SMS; persisted when the user completes that step. Join-team shortcut (address-only path) persists `{ email: true, sms: false }`.

---

## Tests

`app/lib/notifications/resolve-intelligence-notification-prefs.test.ts` — Vitest (`npm test`).

---

## Why `notificationPreferences` is not inside `preferences`

Auth onboarding replaces **`preferences`** with a four-field object when saving units (`auth/lib/firebase/firestore.ts`). A sibling field avoids accidental wipes of intelligence prefs.
