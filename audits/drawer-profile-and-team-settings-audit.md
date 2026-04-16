# Deep audit: `drawer=profile` and `drawer=team-settings`

**Generated:** 2026-04-15  
**Updated:** 2026-04-15 (unified Settings)  
**Codebase:** `C:\Groundzy\app`  
**Purpose:** Single reference for product, engineering, and support on how the Profile and Team Settings drawers are registered, gated, composed, backed by data, and linked from the rest of the app.

---

## 0. Unified Settings (`drawer=settings`) — current product entry

**Primary sidebar destination:** `settings` — label “Settings” (`navigation.drawerLabels.settings`), gear icon, same tier visibility as former Profile (`HOME_PLUS_PRO_AND_TEAMS`).

| Mechanism | Behavior |
|-----------|----------|
| **URL** | `/?drawer=settings&tab=personal` (default) or `tab=organization` |
| **Legacy URLs** | `?drawer=profile` → replaced with `settings` + `tab=personal`; `?drawer=team-settings` → `settings` + `tab=organization` (`components/work-area/work-area-content.tsx`) |
| **Layout** | `app/drawers/settings.tsx`: identity-based switcher (person name vs company name), then `ProfilePanel` or org content |
| **Organization / non-team** | `tab=organization` + non–team tier: value preview + CTA to `upgrade-to-teams`; team tier + `organizationId`: `TeamSettingsPanel`; team tier without org: empty state inside `TeamSettingsPanel` |
| **Panels** | `app/drawers/profile/ProfilePanel.tsx` (personal body); `app/drawers/team-settings/TeamSettingsPanel.tsx` (org body) |
| **Sidebar identity** | `components/navigation/sidebar.tsx`: `SidebarIdentity` block above nav (avatar + name, optional team row) — separate from nav items |
| **Analytics** | `drawer_view` with `drawer_id: settings`; `settings_tab_view` with `tab: personal \| organization` (`lib/firebase/analytics.ts` `logSettingsTabView`) |

`profile` and `team-settings` remain **registered** drawers for redirects and composition; **`visibleInSidebar: false`** on both. Thin wrappers: `profile.tsx` and `team-settings.tsx` wrap panels in `DrawerShell` + `DrawerScrollArea`.

---

## 1. Executive summary

| | **Profile** (`profile`) | **Team settings** (`team-settings`) |
|--|-------------------------|-------------------------------------|
| **User-facing label** | “Profile” (`navigation.drawerLabels.profile`; EN/ES/FR) | “Team” (`navigation.drawerLabels.team-settings`) |
| **Registry** | `lib/drawers.ts` → lazy `@/app/drawers/profile` → `Profile` | `lib/drawers.ts` → lazy `@/app/drawers/team-settings` → `TeamSettings` |
| **Tier visibility (nav + URL guard)** | Home, Plus, Pro, Small/Mid/Large Team, Enterprise (`HOME_PLUS_PRO_AND_TEAMS`) | Small Team, Mid Team, Large Team, Enterprise only |
| **Sidebar / bottom nav** | Visible in sidebar; **not** in bottom nav (`visibleInBottomNav: false`) | Same |
| **Mobile sheet** | `defaultMobileState: "full"` | `defaultMobileState: "full"` |
| **URL** | `/?drawer=profile` (no extra params; all other query keys pass through as `params` if present) | `/?drawer=team-settings` |
| **Primary role** | Personal account: identity, billing, preferences, privacy, usage | Organization: members, invites, logo, shared workflow defaults |

**Important distinction:** Profile is the **individual** user’s settings and subscription. Team settings is the **organization** (`teams/{organizationId}`) tied to `userDoc.organizationId`, with role-based UI (owner/admin vs member/viewer).

---

## 2. URL, registry, and access control

### 2.1 How the query string works

- The app uses **`?drawer=<id>`** on the root path (see `lib/drawer-utils.ts`: `buildDrawerUrl`, `parseDrawerParams`).
- **`drawerIdToUrlParam`** strips a `drawer-` prefix for cleaner URLs; both `profile` and `team-settings` register **without** that prefix, so the param is literally `profile` or `team-settings`.
- **Deep links:** `buildDrawerHref("profile")` → `/?drawer=profile` (see `lib/drawer-utils.ts`).

### 2.2 Tier guard (URL and navigation)

`components/work-area/work-area-content.tsx` defines `canUserAccessDrawer(drawerId, userTier)`:

- Loads `getDrawerMetadata(drawerId).visibleForTiers`.
- If the list is **empty or missing**, access is allowed (no tier restriction).
- Otherwise the user’s effective tier (`getEffectiveSubscriptionTier(userDoc)`) must be **in** that list.

**Profile:** `visibleForTiers` = all consumer + team tiers in `HOME_PLUS_PRO_AND_TEAMS` — so Home through Enterprise (as modeled in `lib/drawers.ts`).

**Team settings:** `visibleForTiers` = `['Small Team', 'Mid Team', 'Large Team', 'Enterprise']` **only**. Pro solo, Plus, and Home users **cannot** open this drawer via URL or nav; if they force `?drawer=team-settings`, the work-area effect **replaces the URL with the path only** (no `drawer` param) and clears the active drawer (`router.replace(pathname ?? "/")`).

**Exception pattern in the same file:** `view-request`, `view-quote`, `view-job`, `view-invoice` are allowed through the tier check when `userTier` is set (record-level access handled inside the drawer). **Profile and team-settings do not use this exception.**

### 2.3 Drawer context (`useDrawer`)

Both components run inside `DrawerProvider` (`lib/drawer-context.tsx`), which exposes:

- `navigate(drawerId, params?)` — updates store + URL via `navigateToDrawer`
- `updateParams` — merge query params for the **current** drawer
- `close` — clear drawer from URL and navigation store

Profile uses `navigate` to open **`my-photos`**, **`tutorial`** targets, upgrade flows, etc. Team settings does not call `navigate` in the current implementation (single-purpose screen).

### 2.4 Analytics

`work-area-content.tsx` logs `drawer_view` to Firebase Analytics when `activeDrawer` changes (includes duration in previous drawer). Both IDs appear as the `activeDrawer` value when opened.

---

## 3. Profile drawer (`profile`)

### 3.1 Entry point and layout

- **File:** `app/drawers/profile.tsx` — exports `Profile`.
- **Shell:** `DrawerShell` + `DrawerScrollArea` from `@/components/drawer-layout` (consistent with other informational drawers).
- **i18n:** `useI18n()` for `t`, `locale`, `setLocale`.

### 3.2 Data sources and hooks

| Concern | Source |
|--------|--------|
| Auth user | `useAuth()` → `user`, `userDoc`, `organization` |
| User doc refresh | `useAuthStore` → `setUserDoc` after Firestore writes |
| Effective tier | `getEffectiveSubscriptionTier(userDoc)` |
| Tier grouping | `isTierInGroup` for home / plus / pro / teams |
| Capabilities | `getCapabilitiesForTier(subscriptionTier)` — used for SMS intelligence toggles (`notifications.sms_critical` is **Teams only** per `lib/capabilities.ts`) |
| Username | `getUsernameFromUserId` (async) for `@username` display |
| Map prefs | `useMapStore` (center, zoom), `useUserPreferences` |
| Properties list | `useProperties(organizationId, { isDeleted: false })` when Plus-only |
| AI / PlantNet usage | `useAiUsage()` |
| Tree limits | `useTreeUsage()` |
| Gallery count | `useGalleryCount()` |
| Plus-only flag | `useIsPlusOnlyProperties()` — true when tier is **Plus only** (properties without clients) |
| Stripe actions | `useProfileStripeActions` |

### 3.3 Section-by-section behavior

Sections are composed from `app/drawers/profile/*.tsx` (and one co-located hook).

1. **`ProfileHeroCard`**
   - Avatar (`UserAvatar`), optional photo upload/remove (`uploadProfilePhoto` / `removeProfilePhoto` from `@/lib/firebase/profile-photo`), display name resolution (Firebase `displayName`, `userDoc.profile`, email local-part fallback).
   - Shows tree usage, AI usage, and photo count summaries with loading states.

2. **`ProfilePlanBillingCard`** (if `user`)
   - Plan label, subscription status, period end, payment failure, beta user paths.
   - CTAs: Plus/Pro checkout, Stripe Customer Portal, “Manage on Stripe”, beta subscribe — all via `useProfileStripeActions` (POST to `/api/stripe/create-checkout-session`, `create-portal-session`, etc.).
   - Progress bars for trees, AI, PlantNet usage; navigates to other drawers where appropriate via `navigate`.

3. **`ProfilePhotosStorageCard`** (if `user`)
   - Opens **`my-photos`** drawer: `navigate("my-photos")`.

4. **`ProfilePropertiesCollapsible`** — **only if** `useIsPlusOnlyProperties()` (Plus tier)
   - Lists properties from `useProperties`; links into property-related flows via `navigate`.

5. **`ProfileMapSettingsCollapsible`**
   - Initial map location: address autocomplete, zoom presets, “use current map”, persistence via `updateUserInitialMapLocation` + `getUser`.
   - GPS-on-load and zoom level UI toggles from `lib/settings-initial-map`.
   - Map marker size preference (`mapMarkerSize` → `setPreferences`).

6. **`ProfileDefaultTreeSettingsCollapsible`**
   - Defaults for new trees: public/private, access requests, feed share — `useUserPreferences` / `setPreferences`.

7. **`ProfileUnitsTimeCollapsible`**
   - Units, temperature, timezone, language (syncs to `setLocale` when language changes), AI chat history retention.

8. **`ProfileWorkItemsCollapsible`**
   - Preview toggles for work-item features (activity timeline, dashboard, dual-write) — preference persistence.

9. **`ProfileTutorialsCollapsible`**
   - `useTutorialState` — dashboard tutorial visibility.

10. **`ProfileIntelligenceNotificationsCollapsible`**
    - Email + SMS for “intelligence” alerts; persists via `updateUserNotificationPreferences` (Firestore).
    - SMS requires **`notifications.sms_critical`** (Teams tiers) **and** a saved phone number; UI disables SMS with reason strings consumed by i18n (`needsTeamsTier` / `needsPhone`).

11. **`ProfilePrivacyCollapsible`**
    - Phone number edit (save/cancel), links that may use `navigate`, privacy-related copy.

12. **`ProfileLogoutSection`**
    - Sign-out affordance at end of scroll.

### 3.4 Firestore / Firebase touchpoints (profile)

- `getUser`, `updateUserPhone`, `updateUserInitialMapLocation`, `updateUserNotificationPreferences`, `getUsernameFromUserId` — `@/lib/firebase/firestore`
- Profile photo — `@/lib/firebase/profile-photo` (Storage + user doc fields)
- Public profile sync for avatars is documented in `lib/firebase/firestore.ts` (e.g. `profile_public` subcollection)

### 3.5 Navigation calls originating from elsewhere **to** Profile

Many drawers call `navigate("profile")` as the **upgrade / fix billing / complete profile** destination, including (non-exhaustive grep): `jobs`, `quotes`, `invoices`, `weather-impact`, `tutorial`, `upgrade-to-teams`, `request-form`, `quote-form`, `job-form`, `invoice-form`, `mapbox-map` (user codes toast), etc.

### 3.6 i18n surface

- Large subtree under `messages.ts` keys: `profile.*` (hero, plan card, usage, intelligence notifications, workflow strings reused in team settings, errors, etc.).
- Navigation label: `navigation.drawerLabels.profile`.

---

## 4. Team settings drawer (`team-settings`)

### 4.1 Entry point and layout

- **File:** `app/drawers/team-settings.tsx` — exports `TeamSettings`.
- **Shell:** `DrawerShell` + `DrawerScrollArea`; destructive actions use `ConfirmDestructiveDialog` + `teamConfirm` state machine.

### 4.2 Preconditions

- **`userDoc.organizationId`** → used as **team document ID** (`teams/{orgId}`).
- If **missing:** early return with copy: not in a team; points users to invite flow on auth domain (“Join a team…”). **No** `useDrawer().navigate` here.

### 4.3 Data loading (client-side Firestore)

On `orgId` + `user` present, `useEffect` loads:

1. **`teams/{orgId}`** — `getDoc` → team name, `logoURL`, `ownerId`, `members`, `memberRoles`, `settings` (e.g. `maxMembers`, invite policy fields).
2. **Each member** — `users/{uid}` for email and profile names.
3. **`invite_codes`** — `where("teamId", "==", orgId)`; filters to `isActive` codes in memory.

Loading and error states are handled inline (spinner header, generic “Failed to load team data”).

### 4.4 Role model

- `userDoc.role` as `TeamRole` — compared to `owner` / `admin` for **`isOwnerOrAdmin`**.
- **Owners/admins** see: logo upload/remove, member role dropdowns, remove member, invite section, workflow settings editor, transfer ownership (owner only).
- **Non-admins** see: read-only logo (if set), leave team; duplicated “Leave Team” in members card for non-admin.

Role options in UI: `admin`, `manager`, `member`, `viewer` (select for non-owner members).

### 4.5 Server actions (`app/actions/team`)

All mutating operations require Firebase ID token from `getCurrentUser()?.getIdToken()`:

| Action | Purpose |
|--------|---------|
| `updateMemberRole` | Change a member’s role |
| `removeMember` | Remove user from team |
| `leaveTeam` | Current user leaves; then refreshes user doc via `getUser` |
| `transferOwnership` | Owner transfers; updates local state + `getUser` |
| `createTeamInviteCode` | New code |
| `revokeInviteCode` | Deactivate code |
| `regeneratePrimaryInviteCode` | New primary code string |

### 4.6 Team logo

- **Upload:** `uploadTeamLogo(orgId, file)` from `@/lib/firebase/team-logo` — invalidates React Query `["teamLogo", orgId]` (used by sidebar `useTeamLogo`).
- **Remove:** `removeTeamLogo` — same invalidation.
- **Validation:** `getAcceptForContext("profile")` for file input accept (same image rules as profile photos per `lib/validate-image`).

### 4.7 Invite codes UX

- List shows code, primary badge, used/max uses, copy, share (`navigator.share` or clipboard fallback), regenerate (primary) or revoke (non-primary).
- Share text uses `getAuthAppUrl()` from `@/lib/auth-redirect` for signup URL.

### 4.8 Workflow settings (owner/admin only)

- **`useWorkflowSettings`** — enabled when `useTeamsOnlyAccess().hasAccess` (workflow pipeline capability) **and** `organizationId` exists.
- Renders **`WorkflowSettingsEditor`** (`components/settings/WorkflowSettingsEditor.tsx`): templates, tax/currency, fees, terms — strings keyed under `profile.workflowSettings.*` in i18n.
- Persisted via `getTeamWorkflowSettings` / `updateTeamWorkflowSettings` in `@/lib/firebase/firestore` (team-scoped document).

**Note:** Workflow settings are **team-level** defaults; the profile drawer does **not** embed this editor — only Team settings (and the hook’s org scoping) does for this UI entry point.

### 4.9 i18n

- **Destructive dialogs** use `t("teamSettings.*", …)` with English fallbacks in `team-settings.tsx`.
- **Card titles** (“Team Info”, “Members”, “Invite Codes”, “Workflow settings”) are largely **hardcoded English** in the component — potential localization gap vs Profile.
- Navigation label: `navigation.drawerLabels.team-settings` → “Team” / “Equipo” / “Equipe”.

### 4.10 Tutorials

`lib/tutorials/definitions.ts` — tutorial id `team-workspace` (Teams-only tier) includes steps with **`trigger: "team-settings"`** for spotlight routing to this drawer.

---

## 5. Sidebar and chrome

### 5.1 Ordering

From `lib/drawers.ts`:

- **`profile`:** `order: 4` — Account section (`MORE_SECTION_LABELS[4] === "Account"`).
- **`my-photos`:** `order: 4.05` — hidden from nav (`hideFromNav: true`).
- **`team-settings`:** `order: 4.051` — explicitly placed after `my-photos`, before `public-profile` (4.1).

### 5.2 Sidebar visuals (`components/navigation/sidebar.tsx`)

- **Profile** nav item: uses `UserAvatar` + `userDoc` / `user`; label shows `@username` or name from `userDoc.profile` / email.
- **Team settings** nav item: uses **`useTeamLogo`** for `teamLogoURL`; **`truncateLabel`** true for long team names.

---

## 6. Related drawers (not duplicates)

| Drawer | Relationship |
|--------|----------------|
| `public-profile` | Other user’s profile; **`requiredProps: userId`**; `hideFromNav`; different component (`public-profile`). |
| `my-photos` | Opened from Profile photos card; `hideFromNav`; not in sidebar. |
| `upgrade-to-teams` | Marketing/checkout flow; may point users toward team features; not a settings screen. |

---

## 7. File inventory (maintenance checklist)

### Profile

| Path | Role |
|------|------|
| `app/drawers/profile.tsx` | Main `Profile` component |
| `app/drawers/profile/ProfileHeroCard.tsx` | Header, avatar, usage |
| `app/drawers/profile/ProfilePlanBillingCard.tsx` | Billing & usage meters |
| `app/drawers/profile/ProfilePhotosStorageCard.tsx` | Link to My Photos |
| `app/drawers/profile/ProfilePropertiesCollapsible.tsx` | Plus-only properties |
| `app/drawers/profile/ProfileMapSettingsCollapsible.tsx` | Map defaults |
| `app/drawers/profile/ProfileDefaultTreeSettingsCollapsible.tsx` | Tree defaults |
| `app/drawers/profile/ProfileUnitsTimeCollapsible.tsx` | Locale/units/time |
| `app/drawers/profile/ProfileWorkItemsCollapsible.tsx` | Work items prefs |
| `app/drawers/profile/ProfileTutorialsCollapsible.tsx` | Tutorial toggles |
| `app/drawers/profile/ProfileIntelligenceNotificationsCollapsible.tsx` | Intelligence email/SMS |
| `app/drawers/profile/ProfilePrivacyCollapsible.tsx` | Privacy + phone |
| `app/drawers/profile/ProfileLogoutSection.tsx` | Log out |
| `app/drawers/profile/useProfileStripeActions.ts` | Stripe API calls |

### Team settings

| Path | Role |
|------|------|
| `app/drawers/team-settings.tsx` | Single-file `TeamSettings` UI + data load |
| `app/actions/team.ts` | Server actions for team mutations (imported by team-settings) |
| `lib/firebase/team-logo.ts` | Logo upload/remove |
| `components/settings/WorkflowSettingsEditor.tsx` | Shared workflow defaults editor |
| `hooks/useWorkflowSettings.ts` | Team workflow settings query + mutation |

### Registry

| Path | Role |
|------|------|
| `lib/drawers.ts` | `registerDrawer("profile", …)` and `registerDrawer("team-settings", …)` |
| `lib/drawer-registry.ts` | Metadata types and nav helpers |
| `lib/i18n/messages.ts` | `navigation.drawerLabels`, `profile.*`, `teamSettings.*` |

---

## 8. Risks, gaps, and test ideas

1. **Team settings copy is mixed EN + i18n** — dialogs use `t()`; many labels are English literals. Align with Profile’s i18n pattern if product requires full localization.
2. **Tier vs org membership** — A user could theoretically have a Teams **tier** without `organizationId` (edge case). UI shows “not in a team” rather than hiding the nav item (nav is tier-based from registry, not org-based).
3. **URL guard** — Non-team tiers cannot deep-link to `team-settings`; they are bounced to `/` without drawer — **silent** from UX (no toast). Consider a one-line toast if product wants explainability.
4. **Invite code optimistic row** — `handleCreateInviteCode` appends a row with `id: "new"`; may duplicate if list refresh patterns change — low risk but worth noting for QA.
5. **Regression tests** — Manual: `/?drawer=profile` on Home/Plus/Pro/Teams; `/?drawer=team-settings` as Pro (should clear drawer); as Small Team with/without `organizationId`; owner vs member invite visibility.

---

## 9. Quick reference URLs

- Profile: `https://app.groundzy.com/?drawer=profile` (or your dev host)
- Team settings: `https://app.groundzy.com/?drawer=team-settings` (Teams tier only; requires org membership for meaningful content)

---

*This document is descriptive of the codebase as of the generation date; behavior is authoritative in `lib/drawers.ts`, `app/drawers/profile.tsx`, `app/drawers/team-settings.tsx`, and `components/work-area/work-area-content.tsx`.*
