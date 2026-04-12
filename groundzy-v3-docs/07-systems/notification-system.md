# Notification system (cross-feature)

**Nature:** **In-app** alerts for a user (e.g. Stripe-driven), distinct from **messaging** (`conversations`) and **email** (`mail` collection in rules).

---

## What exists

| Piece | Location |
|-------|----------|
| **Collection** | `notifications/{notificationId}` |
| **Client API** | `lib/firebase/notifications.ts` — `subscribeNotifications`, `markNotificationRead`, `markAllNotificationsRead` |
| **Type** | `AppNotification` — `userId`, `type`, `title`, `message`, `actionUrl?`, `read`, timestamps |
| **Hooks** | `useNotifications`, `useUnreadNotifications` |
| **UI** | `components/inbox/ActivityTab.tsx` (notification list); badges on **sidebar**, **bottom-nav**, **dashboard** inbox |

**Rules:** User may **read** own notifications; **create** denied from client (`create: if false`); **update** only **`read`** → true (`firebase/firestore.rules`).

**Writers:** Server / Admin (e.g. `lib/stripe-webhook-handlers.ts`, dev routes) — **not** feature drawers.

---

## Duplication & overlap

- **“Inbox”** in product UX often **combines** `notificationsUnread + messagesUnread` (`dashboard.tsx`)—two backends presented as one **badge**.
- **Activity** tab naming overlaps **tree Activity** (history)—three different “activity” meanings (notifications vs messages vs tree events).

---

## Fragmentation

- **`type: string`** on `AppNotification` — **not** a closed enum in the client type—callers must coordinate informally.
- No unified **delivery** layer (push, email, in-app) in the surveyed client module—**in-app only** in this path.

---

## Missing abstraction layers

| Gap | Impact |
|-----|--------|
| **Notification domain service** | Producers (Stripe, future job) each write Firestore shape ad hoc. |
| **User preference** for channel | Not visible in `AppNotification` alone. |
| **Deep link contract** | `actionUrl` free-form—no typed routing to drawer+params. |

---

## v3 foundation direction

- **`Notification`** as stable union type (`billing` \| `workflow` \| `sharing` \| …) + **payload** for structured params.
- **Single inbox facade** that subscribes to notifications + messages + (optional) event feed with **one** read model for UI.
- **Router adapter** maps `actionUrl` or structured payload → app navigation.

---

## Related

- `conversations` / `useUnreadMessages` — parallel messaging system
- `firebase/firestore.rules` § notifications
