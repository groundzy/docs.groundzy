# Drawer refactor: second pass — retrospective & analysis

**Purpose:** Consolidate what was **planned**, what was **implemented**, what remains **optional or missing**, and **recommended** follow-ups. This document synthesizes the **drawer optional second pass** plan (collapsibles, hooks, sidebar/thread splits, view-zone section) together with the overlapping **megafile splits** roadmap (constants, utils, early extractions) and notes from implementation threads.

**Constraint (all passes):** `lib/drawers.ts` must keep stable lazy imports to public entry components (e.g. `Profile`, `RequestForm`, `AiChat`, `ContactUs`, `ViewZone`). New code lives under colocated folders and is imported from existing entry files only.

---

## 1. Executive summary

| Theme | Status |
|-------|--------|
| Profile: Stripe hook + collapsible extractions | **Done** — `useProfileStripeActions` + six `Profile*Collapsible` components; hero/plan/photos cards pre-existed |
| Request form: shared per-item services `Collapsible` | **Done** — `RequestFormItemServicesCollapsible` |
| AI chat: history bar vs thread UI | **Done** — `AiChatSidebar`, `AiChatThread`; utils + `GalleryPickerGrid` also split (megafile phase) |
| Contact-us: thread + composer | **Done** — `ContactUsThread`, `ContactUsThreadComposer`; gallery dialog currently in parent |
| View zone: species + bulk services block | **Done** — `ViewZoneSpeciesAndServicesSection`; dialogs in `ViewZoneDialogs` (earlier megafile phase) |
| Docs: `drawer-audit.md` §3 line counts | **Done** — refreshed after splits |
| **Phase 2 discipline** | **Done** — `npm run verify:drawers` (non-blocking line-count warn), `ContactUsGalleryPickerDialog`, `RequestFormAssigneesSection`, `AiChatComposer` in `ai-chat/AiChatComposer.tsx`, PR checklist + `docs/drawer-shell-classification.md` |
| **Phase C burn-down** (orchestration entry files) | **In progress / ongoing** — sections extracted under colocated folders; targets below reflect `verify:drawers` |
| **`tutorial` DrawerShell compliance** | **Done** — `app/drawers/tutorial.tsx` uses `DrawerShell` + `DrawerScrollArea` on all branches (binary classification rule satisfied). |
| **Nested entry line warnings** | **Done** — `scripts/drawer-pr-check-warn.mjs` scans `NESTED_ENTRY_FILES` (`explore/index.tsx`, `entry-form/index.tsx`, `view-tree/index.tsx`) alongside top-level `app/drawers/*.tsx`. |
| **`entry-form` + `WORKFLOW_DRAWERS`** | **Done** — `entry-form` in `WORKFLOW_DRAWERS`; `useWorkflowFormDirty` + discard dialog + `AddHistoryEntryForm` dirty signaling (`useFormState` / `onDirtyChange`). **Manually re-test** after history-form changes. |
| **Dashboard Phase C pass** | **Done (one pass)** — `DashboardYourTreesSection`, `DashboardMobileDashboardSections` extracted; `dashboard.tsx` still >400 lines but reduced. |
| Optional: `AiChatComposer` as third file | **Done** — `AiChatComposer` lives under `app/drawers/ai-chat/` and is used by `AiChatThread` |
| Optional: request form schedule/assignees | **Done** — `RequestFormAssigneesSection`, `RequestFormTreesAndZonesSection`, `RequestFormScheduleSection`, `RequestFormIntakeSection` |
| Optional: move gallery `Dialog` fully under `ContactUsThread` | **Not done** — shared `ContactUsGalleryPickerDialog` in `contact-us/` for thread + support flows |

**Approximate entry file sizes (Phase C snapshot, `npm run verify:drawers`, line count including blanks):**

| File | Lines (approx.) |
|------|----------------:|
| `view-zone.tsx` | ~778 |
| `request-form.tsx` | ~641 |
| `contact-us.tsx` | ~606 |
| `profile.tsx` | ~615 |
| `ai-chat.tsx` | ~582 |
| `dashboard.tsx` | ~597 (after `DashboardYourTreesSection` + `DashboardMobileDashboardSections` extraction) |

These remain **above** the soft ~400-line orchestration budget; **`view-zone`** is the usual **next** high-yield target (interaction-heavy, still well above budget). Treat **`npm run verify:drawers` output as a ranked burn-down queue**, not only a warning—especially now that **nested** entries (e.g. `view-tree/index.tsx` when >400 lines) appear in CI output.

---

## 2. Plans referenced

### 2.1 Megafile splits (first wave)

Described extracting **pure helpers** first, then **leaf UI**, then **sections**. Concrete items that align with the codebase today:

- **Request form:** `request-form-constants.ts`, `RequestFormMapMarkerThumb.tsx`, import-order fixes.
- **AI chat:** `ai-chat-utils.ts`, `GalleryPickerGrid` under `ai-chat/`.
- **Contact-us:** `contact-us-formatters.ts`, `ParticipantLabel`, `InboxGalleryGrid`.
- **View zone:** zone constants, `ViewZoneDialogs.tsx`.
- **Profile:** section cards (`ProfileHeroCard`, `ProfilePlanBillingCard`, `ProfilePhotosStorageCard`) and phased collapsibles / hook.

### 2.2 Optional second pass (structural)

Focused on **composition** without changing `lib/drawers.ts` entrypoints:

1. `useProfileStripeActions` + presentational **Collapsible** wrappers under `app/drawers/profile/`.
2. `RequestFormItemServicesCollapsible` for duplicated tree/zone service toggles.
3. `AiChatSidebar` + `AiChatThread` (presentational; state stays in `ai-chat.tsx`).
4. `ContactUsThread` + `ContactUsThreadComposer` for the `showThread` branch.
5. `ViewZoneSpeciesAndServicesSection` after dialogs.
6. Optional: refresh `docs/drawer-audit.md` §3.

---

## 3. Delivered artifacts (inventory)

### 3.1 Profile (`app/drawers/profile/`)

| Artifact | Role |
|----------|------|
| `useProfileStripeActions.ts` | Checkout, portal, beta subscribe handlers + loading/error state consumed by `ProfilePlanBillingCard` |
| `ProfilePropertiesCollapsible.tsx` | Plus properties list |
| `ProfileMapSettingsCollapsible.tsx` | Initial map / zoom / GPS |
| `ProfileDefaultTreeSettingsCollapsible.tsx` | Default tree sharing |
| `ProfileUnitsTimeCollapsible.tsx` | Units, temperature, timezone, language |
| `ProfileTutorialsCollapsible.tsx` | Tutorials toggle |
| `ProfilePrivacyCollapsible.tsx` | Phone, security, delete account, policy links |
| Pre-existing | `ProfileHeroCard`, `ProfilePlanBillingCard`, `ProfilePhotosStorageCard` |

`profile.tsx` remains the **composition root** (hooks, `userDoc`-derived flags, i18n `t()`).

### 3.2 Request form

| Artifact | Role |
|----------|------|
| `request-form/RequestFormItemServicesCollapsible.tsx` | Single collapsible for per-item **service type** toggles for both tree and zone rows |
| Pre-existing | `request-form-constants.ts`, `RequestFormMapMarkerThumb.tsx` |

### 3.3 AI chat

| Artifact | Role |
|----------|------|
| `ai-chat/AiChatSidebar.tsx` | “History bar”: new chat, recent chats, delete affordances when history is enabled |
| `ai-chat/AiChatThread.tsx` | Scroll area, messages, suggestions strip; composes `AiChatComposer` |
| `ai-chat/AiChatComposer.tsx` | Composer input, attachments, attach menu |
| `ai-chat/GalleryPickerGrid.tsx` | Gallery picker grid UI |
| `ai-chat/AiChatGalleryPickerDialog.tsx` | “Choose from My Photos” dialog wrapper |
| `ai-chat/AiChatDeleteChatDialog.tsx` | Delete-chat confirmation dialog |
| `ai-chat-utils.ts` | Pure helpers + types (e.g. `aiMessageToChatMessage`, `autoSendKey`) |

### 3.4 Contact-us

| Artifact | Role |
|----------|------|
| `contact-us/ContactUsThread.tsx` | Header, message list, loading state |
| `contact-us/ContactUsThreadComposer.tsx` | Reply-as-support, attachments, input, send |
| `contact-us/ContactUsGalleryPickerDialog.tsx` | Shared gallery picker dialog for thread + support flows |
| `contact-us/ContactUsInboxHome.tsx` | Default inbox: Activity/Messages tabs, admin shortcuts, conversation list |
| `contact-us/ContactUsSupportInbox.tsx`, `ContactUsMessageUserPanel.tsx`, … | Mode-specific panels for support inbox, message-user, share-tree, access requests |
| `contact-us/ContactUsContactSupportPanel.tsx` | Contact Support form + gallery dialog (support flow) |
| Pre-existing | `contact-us-formatters.ts`, `ParticipantLabel.tsx`, `InboxGalleryGrid.tsx` |

**Note:** The gallery picker is a **single** presentational dialog (`ContactUsGalleryPickerDialog`) driven from `contact-us.tsx` so **thread** and **support** attachment flows share one implementation (`galleryPickerTarget === "thread" | "support"`).

### 3.5 View zone

| Artifact | Role |
|----------|------|
| `view-zone/ViewZoneSpeciesAndServicesSection.tsx` | Species aggregates, tree list CTAs, bulk service presets, scheduled zone services list (render-only; mutations stay in `ViewZone`) |
| `view-zone/ViewZoneEditForm.tsx` | Edit mode: name, color, description, property/client, contains list |
| `view-zone/ViewZoneDrawerChrome.tsx` | Header (name, area, edit/delete) + breakout placement strip |
| Pre-existing | `view-zone/ViewZoneDialogs.tsx` |

### 3.6 Dashboard (Phase C pass)

| Artifact | Role |
|----------|------|
| `dashboard/DashboardYourTreesSection.tsx` | “Your trees” block: counts, health grid, trends, hire-a-pro / filter CTAs |
| `dashboard/DashboardMobileDashboardSections.tsx` | Mobile-only upgrade tiles + “All sections” grid |
| Pre-existing | `DashboardUpcomingList`, `DashboardTrendSummary`, `DashboardAiWizardCard`, `DashboardTutorialCard` |

`dashboard.tsx` remains the composition root (weather, hooks, upcoming, quick actions).

---

## 4. Implementation incidents (from threads)

- **`contact-us.tsx` `showThread` branch:** During extraction, JSX briefly ended up **unbalanced** (fragment / ternary not closed before `</DrawerShell>`). The fix was to **replace** the inline thread layout with `<ContactUsThread />` plus the shared gallery `<Dialog />` as siblings inside `DrawerShell`, and to remove **unused** imports (e.g. `ScrollArea`, `Send` duplicated in parent).
- **`lib/drawers.ts`:** No change to lazy import paths — verified against project convention.

---

## 5. Gaps vs plan (honest checklist)

| Item | Plan intent | Current state |
|------|-------------|----------------|
| `AiChatComposer` as separate file | Optional if `AiChatThread` still large | **Done** — `app/drawers/ai-chat/AiChatComposer.tsx` |
| Request form: extract **schedule** or **assignees** | Optional second pass | **Done** — see §3.2 |
| Contact-us: gallery dialog **inside** thread subtree | Prefer one component owning thread UI | **Partially deferred** — shared `ContactUsGalleryPickerDialog` in `contact-us/` |
| i18n for any **new** copy | Use `useI18n` / `t()` for new strings | Mostly **move-only**; hard-coded strings in gallery/dialogs largely pre-existing |
| Automated tests | N/A in plan | **Gap** — no new unit tests for extracted components (recommended for critical paths) |

---

## 6. Recommendations (prioritized)

### 6.1 High value, low risk

1. **`view-zone.tsx`:** Continue Phase C here next—large, interaction-heavy, already partially split under `view-zone/`. Prefer hooks + smaller presentational slices over growing `ViewZone` state in one file.
2. **Orchestration five:** After view-zone, iterate **`request-form`**, **`contact-us`**, **`ai-chat`**, **`profile`** using the same discipline; use `verify:drawers` output as ordering input.
3. **Gallery picker (contact-us):** Either accept the shared dialog in `contact-us.tsx` as the **documented** pattern, or further thin the parent with a router/hook—`ContactUsGalleryPickerDialog` already lives under `contact-us/`.

### 6.2 Medium value

4. **Dashboard:** Further extractions possible (`dashboard.tsx` still >400 lines); weather/quick actions blocks are candidates if churn is high.
5. **Profile:** `profile.tsx` remains a large composition root; **hook composition** (e.g. preferences cluster) if new features add state.

### 6.3 Hygiene

6. Keep **`docs/drawer-audit.md` §3** updated after large PRs (or automate line-count generation in CI as a non-blocking check).
7. **`npx tsc --noEmit`** on touched drawers after refactors; smoke-test: Profile billing, View zone, request create/edit, AI chat history, inbox thread.

---

## 7. Verification checklist (for future PRs)

- [ ] `lib/drawers.ts` lazy imports unchanged for affected drawers.
- [ ] No new user-facing strings without `t()` / messages keys (when adding copy).
- [ ] Drawer layout: `DrawerShell` / `DrawerScrollArea` patterns preserved where used.
- [ ] Thread/contact flows: send message, attachments, gallery picker for thread and support.
- [ ] Profile: Stripe actions still reach `ProfilePlanBillingCard` props correctly.
- [ ] **Entry form (tree history):** when touching `entry-form` or `AddHistoryEntryForm`, smoke-test edit vs create, **Cancel** when dirty, browser refresh, and navigate away.

---

## 8. Related docs

- `docs/drawer-audit.md` — registry inventory, file sizes (§3), cross-cutting notes.
- `docs/DRAWER_PR_CHECKLIST.md` — PR expectations for drawer changes.
- `docs/drawer-navigation.md` — URL / navigation behavior (referenced from `lib/drawers.ts` header).
- `docs/drawer-system-current-state.md` — ground-truth audit of registry, Shell adoption, sizes, workflow drawers.

---

---

## 9. Phase 2 discipline + Phase C (reference)

**Phase 2** added tooling and a few high-leverage extractions: `verify:drawers`, `ContactUsGalleryPickerDialog`, `RequestFormAssigneesSection`, `AiChatComposer`, PR checklist alignment, and `drawer-shell-classification.md` (binary Shell vs documented exception).

**Phase C** is the ongoing **heavy entry file burn-down**: same composition-root pattern (parent keeps orchestration, children are presentational), stable `lib/drawers.ts` imports, and classification doc updates when a drawer **root layout** changes (Shell vs exception). Sub-component extractions that leave `DrawerShell` at the entry root do **not** require a new exception row.

**`verify:drawers` as backlog:** The script is **non-blocking** (exit 0) but the printed list is the practical **next-PR sequence** for Phase C: top-level offenders first, then nested allowlist paths. Add new nested entry files to `NESTED_ENTRY_FILES` in `scripts/drawer-pr-check-warn.mjs` when registering `import("@/app/drawers/.../index")` in `lib/drawers.ts`.

**`entry-form` note:** Scoped `eslint-disable` around `useImperativeHandle` in `add-history-entry-form.tsx` is an intentional trade to avoid dependency churn while exposing `submit` + dirty helpers; re-validate behavior when editing submit handlers or RHF wiring.

---

*Last updated after discipline follow-ups (tutorial Shell, nested verify, entry-form workflow, dashboard extraction pass). Amend when follow-up splits land.*
