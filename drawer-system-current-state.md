# Drawer system — current state (ground truth)

**Generated from repository inspection** (`lib/drawers.ts`, `lib/drawer-registry.ts`, `components/drawer-layout/*`, `app/drawers/**`, docs, `scripts/drawer-pr-check-warn.mjs`, CI).  
**Not** a roadmap; this reflects what exists as of the audit date (last refreshed after discipline follow-ups: `tutorial` Shell compliance, nested `verify:drawers`, `entry-form` workflow dirty, dashboard Phase C extraction pass).

---

## 1. Executive Summary

| Dimension | Assessment |
|-----------|------------|
| **Maturity** | **Structured and partially enforced**: shared layout primitives (`DrawerShell`, `DrawerScrollArea`, etc.), a documented binary rule + exception table, CI that runs a **non-blocking** line-count warning (`verify:drawers`) on **top-level and allowlisted nested** entry files. |
| **Strengths** | Central `registerDrawer` + lazy imports in `lib/drawers.ts`; metadata-driven nav; retry on dynamic imports; colocated folders for hot paths; **all** registered drawers now match **DrawerShell or documented exception**; tree history **`entry-form`** participates in **`WORKFLOW_DRAWERS`** with dirty state + discard dialog. |
| **Risks** | Many entry files remain **>400 lines** (Phase C soft budget). Duplicate **numeric** `order` values (`1.1`, etc.) can make nav sort order sensitive. **`entry-form`** dirty path should be **manually regression-tested** (edit/create, cancel when dirty, refresh, navigate away) after changes to `AddHistoryEntryForm` / navigation. |

**Honest assessment:** The Shell/classification binary rule is **fully satisfied in code** (no unclassified roots). Line-count warnings are a **working burn-down queue**, not just noise—especially now that nested entries (e.g. `view-tree/index.tsx`) surface in CI output. Phase C continues on orchestration-heavy files (`view-zone`, `request-form`, `contact-us`, `ai-chat`, `profile` are the highest-yield next targets).

---

## 2. Drawer system architecture (current)

### URL → registry → entry → UI

1. **Navigation / URL** sync drives `activeDrawer` and params (see `components/work-area/work-area-content.tsx` and `stores/navigation-store.ts`).
2. **`lib/drawers.ts`** calls **`registerDrawer`** for each drawer id: attaches **metadata** (label, icon, `order`, tiers, `hideFromNav`, `defaultMobileState`, `role`, `category`, etc.) and a **lazy import** of the entry module’s exported component.
3. **`lib/drawer-registry.ts`** stores id → `{ metadata, component: lazy(...) }`. **`getDrawerComponent(id)`** returns the lazy component; **`getDrawerMetadata`** / **`getAllDrawerMetadata`** power sidebar, bottom nav, and More grouping.
4. **Work area** resolves `drawerId` and renders the lazy component with parsed props. **Entry files** are responsible for composing drawer UX: shell, scroll region, headers/footers, and wiring data — not for registering themselves (registration is only in `lib/drawers.ts`).

### Role of `lib/drawers.ts`

- **Single registration source** for all drawers (53 `registerDrawer` calls).
- **`withRetry`** on imports mitigates `ChunkLoadError` after cache clears / HMR.
- **Comments** describe nav intent (e.g. fractional `order`, tier visibility); **`MORE_SECTION_LABELS`** / **`NAV_GROUP_LABELS`** align with grouped nav.

### Loading and rendering

- Components are **`React.lazy`**-loaded per id; the work area mounts the active drawer (and related caching/preload behavior in `work-area-content.tsx`).
- **Default mobile sheet height** can fall back via **`getEffectiveDefaultMobileState`** when `defaultMobileState` is omitted (`drawer-registry.ts` uses `role`).

### Entry files

- **Default export convention:** each dynamic import targets the named export used in `lib/drawers.ts` (e.g. `{ default: m.Dashboard }`).
- Some logical drawers share one module (e.g. `clients`, `properties`, `clients-properties` → `ClientsProperties`).

---

## 3. DrawerShell adoption status

**Classification rule (from `docs/drawer-shell-classification.md`):** each registered id is either **DrawerShell** (or an approved wrapper that renders it, e.g. `view-tree` → `ViewTreeDrawerLayout`) **or** a **documented exception**.

**Documented exceptions** (approved alternative patterns): `search`, `tree-add`, `edit-tree`, `multiple-add`, `ai-identifying-wand`, `groundzy-wizard`.

**Counts (53 registered ids):**

| Category | Count |
|----------|------:|
| DrawerShell directly | 46 |
| Wrapper using DrawerShell (`view-tree` → `ViewTreeDrawerLayout`) | 1 |
| Documented exceptions | 6 |
| **Not** DrawerShell and **not** in exception doc | **0** |

(46 + 1 + 6 = 53.)

### Scroll / layout notes

- Repo-wide grep for `overflow-y-auto` under `app/drawers` shows **small localized regions** (e.g. max-height lists in `AiChatSidebar`, assignee picker, tree cards) — not a full primary-column duplicate scroll. No systematic violation surfaced from that pass.
- **Risk:** drawers that omit `DrawerScrollArea` for the main column may still use ad hoc scrolling; this audit did not mechanically prove full compliance per file.

### Full registry table

| Drawer ID | Entry file (import path) | Layout type | Status |
|-----------|---------------------------|---------------|--------|
| `dashboard` | `app/drawers/dashboard.tsx` | DrawerShell | ✅ |
| `weather` | `app/drawers/weather-impact.tsx` | DrawerShell | ✅ |
| `trees` | `app/drawers/trees.tsx` | DrawerShell | ✅ |
| `tree-add` | `app/drawers/tree-add.tsx` | Thin host + `AddTreeForm` | ⚠️ documented |
| `multiple-add` | `app/drawers/multiple-add.tsx` | `DrawerBody` / `DrawerFooter` map workflow | ⚠️ documented |
| `search` | `app/drawers/search.tsx` | `FilterBar` + `ListContainer` | ⚠️ documented |
| `view-tree` | `app/drawers/view-tree/index.tsx` | `ViewTreeDrawerLayout` → DrawerShell | ✅ |
| `entry-form` | `app/drawers/entry-form/index.tsx` | DrawerShell | ✅ |
| `restricted-tree` | `app/drawers/restricted-tree.tsx` | DrawerShell | ✅ |
| `edit-tree` | `app/drawers/edit-tree.tsx` | Thin host + `AddTreeForm` | ⚠️ documented |
| `view-zone` | `app/drawers/view-zone.tsx` | DrawerShell | ✅ |
| `clients-properties` | `app/drawers/clients-properties.tsx` | DrawerShell | ✅ |
| `clients` | `app/drawers/clients-properties.tsx` (same module) | DrawerShell | ✅ |
| `view-client` | `app/drawers/view-client.tsx` | DrawerShell | ✅ |
| `add-client` | `app/drawers/client-form.tsx` | DrawerShell | ✅ |
| `edit-client` | `app/drawers/client-form.tsx` | DrawerShell | ✅ |
| `properties` | `app/drawers/clients-properties.tsx` (same module) | DrawerShell | ✅ |
| `view-property` | `app/drawers/view-property.tsx` | DrawerShell | ✅ |
| `add-property` | `app/drawers/property-form.tsx` | DrawerShell | ✅ |
| `edit-property` | `app/drawers/property-form.tsx` | DrawerShell | ✅ |
| `requests` | `app/drawers/requests.tsx` | DrawerShell | ✅ |
| `view-request` | `app/drawers/view-request.tsx` | DrawerShell | ✅ |
| `add-request` | `app/drawers/request-form.tsx` | DrawerShell | ✅ |
| `edit-request` | `app/drawers/request-form.tsx` | DrawerShell | ✅ |
| `quotes` | `app/drawers/quotes.tsx` | DrawerShell | ✅ |
| `view-quote` | `app/drawers/view-quote.tsx` | DrawerShell | ✅ |
| `add-quote` | `app/drawers/quote-form.tsx` | DrawerShell | ✅ |
| `edit-quote` | `app/drawers/quote-form.tsx` | DrawerShell | ✅ |
| `jobs` | `app/drawers/jobs.tsx` | DrawerShell | ✅ |
| `view-job` | `app/drawers/view-job.tsx` | DrawerShell | ✅ |
| `add-job` | `app/drawers/job-form.tsx` | DrawerShell | ✅ |
| `edit-job` | `app/drawers/job-form.tsx` | DrawerShell | ✅ |
| `invoices` | `app/drawers/invoices.tsx` | DrawerShell | ✅ |
| `view-invoice` | `app/drawers/view-invoice.tsx` | DrawerShell | ✅ |
| `add-invoice` | `app/drawers/invoice-form.tsx` | DrawerShell | ✅ |
| `edit-invoice` | `app/drawers/invoice-form.tsx` | DrawerShell | ✅ |
| `profile` | `app/drawers/profile.tsx` | DrawerShell | ✅ |
| `public-profile` | `app/drawers/public-profile.tsx` | DrawerShell | ✅ |
| `my-photos` | `app/drawers/my-photos.tsx` | DrawerShell | ✅ |
| `hire-groundzy-pro` | `app/drawers/hire-groundzy-pro.tsx` | DrawerShell | ✅ |
| `upgrade-to-plus` | `app/drawers/upgrade-to-plus.tsx` | DrawerShell | ✅ |
| `upgrade-to-pro` | `app/drawers/upgrade-to-pro.tsx` | DrawerShell | ✅ |
| `upgrade-to-teams` | `app/drawers/upgrade-to-teams.tsx` | DrawerShell | ✅ |
| `explore` | `app/drawers/explore/index.tsx` | DrawerShell | ✅ |
| `ai-identifying-wand` | `app/drawers/ai-identifying-wand.tsx` | Full-bleed wizard host | ⚠️ documented |
| `ai-chat` | `app/drawers/ai-chat.tsx` | DrawerShell | ✅ |
| `groundzy-wizard` | `app/drawers/groundzy-wizard.tsx` | Custom tab shell + embedded flows | ⚠️ documented |
| `tutorial` | `app/drawers/tutorial.tsx` | DrawerShell + `DrawerScrollArea` | ✅ |
| `help` | `app/drawers/help.tsx` | DrawerShell | ✅ |
| `contact-us` | `app/drawers/contact-us.tsx` | DrawerShell | ✅ |
| `team-settings` | `app/drawers/team-settings.tsx` | DrawerShell | ✅ |
| `draw` | `app/drawers/draw.tsx` | DrawerShell | ✅ |
| `measure` | `app/drawers/measure.tsx` | DrawerShell | ✅ |

---

## 4. Entry file size analysis

### Top-level `app/drawers/*.tsx` (line counts)

Counts from `npm run verify:drawers` (includes blanks). **Treat the script output as the working burn-down queue** for Phase C: same files tend to stay “hot” until extracted.

| Lines | File | >400? |
|------:|------|-------|
| 843 | `client-form.tsx` | Yes |
| 778 | `view-zone.tsx` | Yes |
| 759 | `trees.tsx` | Yes |
| 752 | `team-settings.tsx` | Yes |
| 748 | `quote-form.tsx` | Yes |
| 692 | `invoice-form.tsx` | Yes |
| 671 | `job-form.tsx` | Yes |
| 659 | `hire-groundzy-pro.tsx` | Yes |
| 641 | `request-form.tsx` | Yes |
| 631 | `view-request.tsx` | Yes |
| 626 | `weather-impact.tsx` | Yes |
| 615 | `profile.tsx` | Yes |
| 606 | `contact-us.tsx` | Yes |
| 597 | `dashboard.tsx` | Yes |
| 582 | `ai-chat.tsx` | Yes |
| 523 | `view-property.tsx` | Yes |
| 435 | `upgrade-to-teams.tsx` | Yes |
| 430 | `property-form.tsx` | Yes |
| 410 | `clients-properties.tsx` | Yes |
| 403 | `multiple-add.tsx` | Yes |

**Top 20** are the rows above (next largest below budget: `view-client.tsx` ~367 lines).

### Nested entry files (`verify:drawers` second pass)

[`scripts/drawer-pr-check-warn.mjs`](../scripts/drawer-pr-check-warn.mjs) **`NESTED_ENTRY_FILES`** — keep in sync when adding `import("@/app/drawers/.../index")` in [`lib/drawers.ts`](../lib/drawers.ts).

| Lines | File | >400? |
|------:|------|-------|
| 558 | `app/drawers/view-tree/index.tsx` | Yes |
| 289 | `app/drawers/entry-form/index.tsx` | No (workflow dirty wiring added; still under budget) |
| 92 | `app/drawers/explore/index.tsx` | No |

### Orchestration vs UI heaviness

- **Still UI-heavy / state-heavy at root:** `contact-us.tsx`, `view-zone.tsx`, `ai-chat.tsx`. **`dashboard.tsx`** had a **Phase C pass**: `DashboardYourTreesSection` + `DashboardMobileDashboardSections` extracted under [`app/drawers/dashboard/`](../app/drawers/dashboard/); file remains >400 lines but is smaller than before.
- **Closer to “clean” composition roots:** `request-form.tsx` and `profile.tsx` — large but mostly **imported sections** and hooks; `explore/index.tsx` is already small.
- **Forms:** `client-form`, `quote-form`, `invoice-form`, `job-form` remain long partly due to **react-hook-form + mutations + workflow dirty** in one file.

---

## 5. Extraction coverage (required drawers)

### `dashboard` (Phase C pass completed once)

| Extracted / colocated | Role |
|----------------------|------|
| `dashboard/DashboardYourTreesSection.tsx` | “Your trees” stats, health, trends, CTAs |
| `dashboard/DashboardMobileDashboardSections.tsx` | Mobile-only upgrade grid + “All sections” nav |
| Pre-existing | `DashboardUpcomingList`, `DashboardTrendSummary`, `DashboardAiWizardCard`, `DashboardTutorialCard`, etc. |

**Entry file:** still orchestrates weather, hooks, greeting, upcoming, AI wizard card, plan card, quick actions, desktop layout.

### `contact-us`

| Extracted / colocated | Role |
|----------------------|------|
| `contact-us/ContactUsThread.tsx`, `ContactUsThreadComposer.tsx`, `ContactUsInboxHome.tsx`, `ContactUsSupportInbox.tsx`, `ContactUsMessageUserPanel.tsx`, `ContactUsShareTreePanel.tsx`, `ContactUsAccessRequestDetailPanel.tsx`, `ContactUsAccessRequestsListPanel.tsx`, `ContactUsContactSupportPanel.tsx`, `ContactUsGalleryPickerDialog.tsx`, `InboxGalleryGrid.tsx`, `ParticipantLabel.tsx` | Panels, thread, gallery |
| `contact-us-formatters.ts` | Time/date helpers |

**Still large inline in `contact-us.tsx`:** orchestration (params → view), hooks for conversations/access requests, attachment/gallery picker state, multiple top-level `DrawerShell` branches.

**Next extractions:** a single **router/view switch** component (props-in → rendered panel); optional hook `useContactUsRouteState` to shrink the entry file.

### `request-form`

| Extracted | Role |
|-----------|------|
| `request-form/RequestFormAssigneesSection.tsx`, `RequestFormClientPropertySection.tsx`, `RequestFormIntakeSection.tsx`, `RequestFormRequestDetailsSection.tsx`, `RequestFormScheduleSection.tsx`, `RequestFormTreesAndZonesSection.tsx`, `RequestFormItemServicesCollapsible.tsx` | Sections |
| `request-form-constants.ts` | Shared constants |

**Entry file:** mostly wiring, `useWorkflowFormDirty`, map/zoom hooks, section composition.

**Next extractions:** custom hooks (`useRequestFormState`, `useRequestMapIntegration`) if the file grows again.

### `view-zone`

| Extracted | Role |
|-----------|------|
| `view-zone/ViewZoneDialogs.tsx`, `ViewZoneDrawerChrome.tsx`, `ViewZoneEditForm.tsx`, `ViewZoneSpeciesAndServicesSection.tsx` | Chrome, dialogs, edit, species/services |

**Still heavy in `view-zone.tsx`:** data hooks, edit/breakout/schedule state, mutations, property/client linkage.

**Next extractions:** `useViewZoneEditingState` / `useViewZoneScheduleFlow` or move remaining JSX blocks matching `ViewZoneEditForm` patterns.

### `ai-chat`

| Extracted | Role |
|-----------|------|
| `ai-chat/AiChatSidebar.tsx`, `AiChatThread.tsx`, `AiChatComposer.tsx`, `AiChatGalleryPickerDialog.tsx`, `AiChatDeleteChatDialog.tsx`, `GalleryPickerGrid.tsx` | UI chunks |
| `ai-chat-utils.ts` (sibling of entry) | Message mapping helpers |

**Entry file:** Firebase subscriptions, send/delete flows, attachment/gallery state, `DrawerShell` + sidebar/thread.

**Next extractions:** `useAiChatController` hook for network/state; keep `ai-chat.tsx` as thin shell.

### `profile`

| Extracted | Role |
|-----------|------|
| `profile/ProfileHeroCard.tsx`, `ProfilePlanBillingCard.tsx`, `ProfilePhotosStorageCard.tsx`, collapsibles (map, units, tutorials, privacy, properties, default tree), `ProfileLogoutSection.tsx`, `useProfileStripeActions.ts` | Cards / sections |

**Still in `profile.tsx`:** phone/username, locale, map prefs, many `useState` fields and save handlers.

**Next extractions:** `useProfileSettings` or split “account” vs “preferences” entries if desired (would be a larger product split).

---

## 6. Enforcement and discipline status

| Artifact | Present | Role |
|----------|---------|------|
| `docs/DRAWER_PR_CHECKLIST.md` | Yes | Human checklist (Shell vs exception, scroll, registry, `role`, workflow dirty, i18n). |
| `docs/drawer-shell-classification.md` | Yes | Binary rule + **exception table** + Phase C line budget note. |
| `scripts/drawer-pr-check-warn.mjs` | Yes | Warns on top-level `app/drawers/*.tsx` **and** allowlisted nested entries (`NESTED_ENTRY_FILES`) **> 400 lines**; **exit 0 always**. |
| `npm run verify:drawers` | Yes in `package.json` | Runs the script above. |
| `.github/workflows/ci.yml` | Yes | Runs **`npm run verify:drawers`** after lint, **before build** — still **warning-only** (script never fails). |

**Are rules followed in practice?** Yes for the **binary Shell vs exception** rule (no unclassified drawer roots at time of refresh). Scroll rules are **not** mechanically linted.

**Violations?** Line count only; no ESLint gate on DrawerShell.

**Enforcement:** **Symbolic for CI** (no failing build); **actionable as a queue** — `verify:drawers` output is the practical Phase C backlog (top-level + nested warnings).

---

## 7. Registry integrity check

- **Registered drawer count:** **53** `registerDrawer` calls in `lib/drawers.ts`.
- **Duplicate `order` values (intentional grouping / collision):**
  - **`order: 1.1`:** `trees`, `ai-identifying-wand`, `groundzy-wizard`, `contact-us` — **high collision**; nav relies on **secondary sort** (stable ordering of filtered lists) and **category** / **bottomNavPriority** where applicable. Worth watching if nav order looks “random.”
  - **`order: 4.05`:** `my-photos`, `team-settings` — same fractional order; both use distinct ids and metadata.
- **Multi-ID → one module:** `clients`, `properties`, `clients-properties` share `ClientsProperties` with `getClientsPropertiesMode` (`clients-properties-mode.ts`) — **structured**. Add/edit pairs (`client-form`, `property-form`, etc.) use **`effectiveDrawerId`** strings for workflow dirty.
- **Misleading / stale:** `drawer-registry.ts` header still says “Supports 20+ drawers” — **undercounts** current registry.
- **Ambiguity:** `registerDrawer` allows duplicate lazy entries for the same module; last write wins in the `Map` — **not** an issue if each id is registered once (current code registers each id once).

---

## 8. Workflow consistency (`WORKFLOW_DRAWERS` + dirty state)

**Definition** (`stores/navigation-store.ts`): `WORKFLOW_DRAWERS` lists drawers that trigger **unsaved changes** confirmation when navigating away with **`workflowFormDirty`**.

**Current set:**  
`add-request`, `edit-request`, `add-quote`, `edit-quote`, `add-job`, `edit-job`, `add-invoice`, `edit-invoice`, `add-client`, `edit-client`, `add-property`, `edit-property`, **`entry-form`**.

**Hook usage:** `useWorkflowFormDirty` appears in **`request-form`**, **`quote-form`**, **`job-form`**, **`invoice-form`**, **`client-form`**, **`property-form`**, **`entry-form`** (`entry-form/index.tsx`). Tree history form dirty state is driven from **`AddHistoryEntryForm`** via `useFormState` + `onDirtyChange` + optional `getIsDirty` on the imperative handle (see `components/trees/add-history-entry-form.tsx`).

**Manual regression (recommended after changes to history forms or navigation):** edit existing entry, create new entry, **Cancel** when dirty, browser refresh/close, navigate away from drawer.

**Gaps / inconsistency:**

| Drawer / surface | In `WORKFLOW_DRAWERS`? | Notes |
|------------------|------------------------|--------|
| `draw` (zone polygon) | No | Map tool; separate concerns (`multipleAddCloseRequested` etc.). |
| `upgrade-to-teams` | No | May collect input; not wired to workflow dirty. |

**Conclusion:** CRM-style workflow forms and **`entry-form`** share discard protection; other form-like surfaces are intentionally out of scope unless added later.

---

## 9. Known gaps and technical debt

| Area | Detail |
|------|--------|
| **Megafiles (top-level + nested)** | ~20 top-level files **>400 lines** plus **`view-tree/index.tsx`** (nested); Phase C ongoing. **Next high-yield orchestration targets:** `view-zone`, `request-form`, `contact-us`, `ai-chat`, `profile` (`view-zone` often cited as largest interaction-heavy remaining). |
| **Registry comment drift** | `drawer-registry.ts` header still says “20+ drawers” — undercounts. |
| **Order collision** | Four drawers share `order: 1.1` — watch if sidebar/More order looks wrong. |
| **entry-form complexity** | Dirty path crosses **`AddHistoryEntryForm`** imperative API + scoped `eslint-disable` around `useImperativeHandle` — acceptable trade; re-test when touching submits or RHF wiring. |

---

## 10. System health score (1–10)

| Criterion | Score | Justification |
|-----------|------:|---------------|
| **Architecture** | **8** | Clear registry, lazy loading, metadata-driven nav, shared layout components. |
| **Consistency** | **7** | Shell binary rule satisfied; workflow dirty covers CRM forms + `entry-form`; remaining gaps are intentional (map tools, etc.). |
| **Maintainability** | **6** | Many entry files still oversized; `verify:drawers` now tracks nested entries—better backlog visibility. |
| **Scalability** | **7** | Registry pattern scales; watch duplicate `order` and file size growth. |

---

## 11. Recommended next actions (max 5, evidence-based)

1. **Phase C — `view-zone.tsx` next:** Largest interaction-heavy orchestration file among the priority five; extract hooks (`useViewZone*`) and/or thin `view-zone.tsx` further using existing `view-zone/*` sections.
2. **Phase C — same discipline on** `request-form`, `contact-us`, `ai-chat`, `profile` — follow `verify:drawers` ordering as a **ranked queue** (or rank by product churn × file size).
3. **Refresh `drawer-registry.ts` header** to reflect **50+** registered drawers (small hygiene).
4. **Optional:** Reduce **`order: 1.1`** collision (four ids) if nav ordering bugs appear in QA.
5. **When touching tree history:** Re-run **manual `entry-form`** checks (edit, create, dirty cancel, refresh, navigate away).

---

## Related docs

- [`drawer-refactor-second-pass-retrospective.md`](drawer-refactor-second-pass-retrospective.md) — second-pass deliverables, Phase C notes, `verify:drawers` as backlog.
- [`DRAWER_PR_CHECKLIST.md`](DRAWER_PR_CHECKLIST.md), [`drawer-shell-classification.md`](drawer-shell-classification.md), [`drawer-navigation.md`](drawer-navigation.md)

---

## Appendix: `components/drawer-layout` inventory

| File | Purpose |
|------|---------|
| `drawer-shell.tsx` | Flex column, `min-h-0`, **no** outer scroll |
| `drawer-scroll-area.tsx` | Primary scroll region (`overflow-y-auto`) |
| `drawer-body.tsx`, `drawer-section.tsx`, `drawer-header.tsx`, `drawer-footer.tsx`, `drawer-form-section.tsx` | Composition |
| `filter-bar.tsx`, `list-container.tsx` | List/search surfaces |
| `index.ts` | Re-exports |
