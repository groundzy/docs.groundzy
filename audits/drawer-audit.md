# Full drawer audit — Groundzy (`app.groundzy`)

**Generated:** 2026-03-26  
**Scope:** `lib/drawers.ts`, `lib/drawer-registry.ts`, `lib/drawer-context.tsx`, `lib/drawer-utils.ts`, `components/work-area/work-area-content.tsx`, `stores/navigation-store.ts`, `app/drawers/**`

This document satisfies the audit brief: registry inventory, wiring, per-drawer notes (grouped where repetitive), cross-cutting issues, and deliverables A–D.

---

## 1. Registry & wiring

### 1.1 `DrawerMetadata` and helpers (`lib/drawer-registry.ts`)

- **Types:** `DrawerMetadata<T>` includes `id`, `label`, `icon`, `order`, `category`, `defaultProps`, `requiredProps`, `visibleForTiers`, `visibleInSidebar`, `visibleInMoreOnly`, `visibleInBottomNav`, `bottomNavPriority`, `hideFromNav`, `defaultMobileState`, `extendOverBottomNav`, `archetype`.
- **Nav helpers:** `getSidebarDrawerMetadata`, `getBottomNavDrawerMetadata`, `getMoreDrawerMetadata`, `getMoreDrawerGroups`, `getDashboardNavItemsForMobile`, `getUpgradeOptionsForTier`, `getUpgradeGlowClass`.
- **Section labels:** `MORE_SECTION_LABELS` and `NAV_GROUP_LABELS` live in `lib/drawers.ts` (not the registry file).

### 1.2 `lib/drawers.ts` — `withRetry`

- **`withRetry`** wraps dynamic import for **`entry-form`** only (retries on `ChunkLoadError` / HMR). Other drawers use plain `() => import(...)`.

### 1.3 How drawers open / close (evidence)

| Mechanism | Location | Role |
|-----------|----------|------|
| URL as source of truth | `components/work-area/work-area-content.tsx` | `parseDrawerParams` + `useEffect` sync: sets `activeDrawer`, `workAreaState`, `mobileWorkAreaState`, `syncHistoryFromUrl`; tier guard `canUserAccessDrawer`; initial `/` → `dashboard` |
| Drawer UI + lazy load | `work-area-content.tsx` | `DrawerWrapper` + `Suspense` + `getDrawerComponent`; imports `@/lib/drawers` to register all drawers |
| Context inside drawer | `lib/drawer-context.tsx` | `DrawerProvider` supplies `navigate`, `updateParams`, `close` (pushes `/?` with `drawer` query) |
| History stack (URL sync / optimistic params) | `stores/navigation-store.ts` | `drawerHistory`, `pushToHistory`, `syncHistoryFromUrl`; `attemptWorkflowNavigate` for dirty **workflow** drawers |
| Workflow dirty set | `navigation-store.ts` | `WORKFLOW_DRAWERS`: request/quote/job/invoice add+edit only (not client/property forms) |
| Mobile chrome | `work-area-mobile.tsx`, `work-area-desktop.tsx` | Title, sheet controls; back/forward is **browser** only |
| Special mobile rules | `work-area-content.tsx` | e.g. `draw` drawer may open with sheet `closed` when no pending polygon on narrow viewports |

### 1.4 Work-area header

- Work-area chrome shows the drawer title and layout controls (expand/dock, close, etc.). **Back/forward** is not duplicated in the header; users rely on **browser** history.

---

## 2. Complete `registerDrawer` inventory (56 drawer IDs)

Each row: **id** → **import path** → **notable metadata** (omitted keys use registry defaults).

| id | Lazy import | Key metadata |
|----|-------------|--------------|
| `dashboard` | `@/app/drawers/dashboard` → `Dashboard` | archetype informational; `visibleForTiers` HOME+…; S+B; `bottomNavPriority` 1; `defaultMobileState` full |
| `weather` | `@/app/drawers/weather-impact` → `WeatherImpact` | informational; `visibleInSidebar` false, `visibleInBottomNav` false; order 1.5 |
| `trees` | `@/app/drawers/trees` → `Trees` | list; S+B; `bottomNavPriority` 2; order 1.1 |
| `tree-add` | `@/app/drawers/tree-add` → `TreeAdd` | form; `hideFromNav` true; `extendOverBottomNav` false |
| `multiple-add` | `@/app/drawers/multiple-add` → `MultipleAdd` | form; `hideFromNav`; `defaultMobileState` half; **`extendOverBottomNav` true** |
| `search` | `@/app/drawers/search` → `Search` | informational; `hideFromNav`; `extendOverBottomNav` false |
| `view-tree` | `@/app/drawers/view-tree/index` → `ViewTree` | detail; **`requiredProps` treeId**; `hideFromNav`; `extendOverBottomNav` false |
| `entry-form` | `@/app/drawers/entry-form` → `EntryForm` | form; **`withRetry`**; **`requiredProps` treeId, entryType**; optional entryId, returnTo |
| `restricted-tree` | `@/app/drawers/restricted-tree` → `RestrictedTree` | detail; **`requiredProps` treeId**; `defaultMobileState` half |
| `edit-tree` | `@/app/drawers/edit-tree` → `EditTree` | form; **`requiredProps` treeId**; **`extendOverBottomNav` true** |
| `view-zone` | `@/app/drawers/view-zone` → `ViewZone` | detail; **`requiredProps` zoneId**; `defaultMobileState` half |
| `clients-properties` | `@/app/drawers/clients-properties` → `ClientsProperties` | list; **`category` workflow**; Pro+ tiers; S+B; `bottomNavPriority` 4 |
| `clients` | same component | list; **`category` data**; Pro+; **`hideFromNav` true**; **`visibleInSidebar` false** |
| `view-client` | `@/app/drawers/view-client` → `ViewClient` | detail; **`requiredProps` clientId**; `defaultMobileState` half |
| `add-client` | `@/app/drawers/client-form` → `ClientForm` | form; `hideFromNav`; **`extendOverBottomNav` true** |
| `edit-client` | same | form; **`requiredProps` clientId**; `extendOverBottomNav` true |
| `properties` | same as `clients-properties` | list; **`category` data**; Pro+; **`hideFromNav`**; **`visibleInSidebar` false** |
| `view-property` | `@/app/drawers/view-property` → `ViewProperty` | detail; **`requiredProps` propertyId**; half |
| `add-property` | `@/app/drawers/property-form` → `PropertyForm` | form; `hideFromNav`; **`extendOverBottomNav` true** |
| `edit-property` | same | form; **`requiredProps` propertyId**; `extendOverBottomNav` true |
| `requests` | `@/app/drawers/requests` → `Requests` | list; **`category` workflow**; **Teams-only** `visibleForTiers`; S; **`visibleInBottomNav` false** |
| `view-request` | `@/app/drawers/view-request` → `ViewRequest` | detail; **`requiredProps` requestId**; Teams-only; half |
| `add-request` | `@/app/drawers/request-form` → `RequestForm` | form; Teams; **`extendOverBottomNav` true** |
| `edit-request` | same | form; **`requiredProps` requestId**; Teams; `extendOverBottomNav` true |
| `quotes` | `@/app/drawers/quotes` → `Quotes` | list; workflow; Teams; S; **not** bottom nav |
| `view-quote` | `@/app/drawers/view-quote` → `ViewQuote` | detail; Teams; **`requiredProps` quoteId** |
| `add-quote` / `edit-quote` | `@/app/drawers/quote-form` → `QuoteForm` | form; Teams; edit has **`requiredProps` quoteId** |
| `jobs` | `@/app/drawers/jobs` → `Jobs` | list; workflow; Teams; S; not bottom nav |
| `view-job` / `add-job` / `edit-job` | `@/app/drawers/job-form` → `JobForm` for forms | same pattern as quotes |
| `invoices` | `@/app/drawers/invoices` → `Invoices` | list; Teams; S; not bottom nav |
| `view-invoice` / add / edit | `@/app/drawers/invoice-form` → `InvoiceForm` | Teams; **`requiredProps` invoiceId** on view/edit |
| `profile` | `@/app/drawers/profile` → `Profile` | informational; S; **`visibleInBottomNav` false** |
| `public-profile` | `@/app/drawers/public-profile` → `PublicProfile` | detail; **`requiredProps` userId**; `hideFromNav` |
| `my-photos` | `@/app/drawers/my-photos` → `MyPhotos` | informational; **`hideFromNav`**; **`visibleInSidebar` false**; **`visibleInBottomNav` false** |
| `hire-groundzy-pro` | `@/app/drawers/hire-groundzy-pro` → `HireGroundzyPro` | informational; **`visibleForTiers` Home, Plus**; S; not bottom nav |
| `upgrade-to-plus` | `@/app/drawers/upgrade-to-plus` → `UpgradeToPlus` | informational; **`category` upgrades**; **Home only** |
| `upgrade-to-pro` | `@/app/drawers/upgrade-to-pro` → `UpgradeToPro` | informational; upgrades; **Home, Plus** |
| `upgrade-to-teams` | `@/app/drawers/upgrade-to-teams` → `UpgradeToTeams` | **archetype form**; Home, Plus, Pro; **`extendOverBottomNav` true** |
| `explore` | `@/app/drawers/explore` → `Explore` | informational; **`category` features**; S+B; `bottomNavPriority` 3 |
| `ai-identifying-wand` | `@/app/drawers/ai-identifying-wand` → `AiIdentifyingWand` | features; S; not bottom nav; **`showHeaderNav` true** (see §1.4) |
| `ai-chat` | `@/app/drawers/ai-chat` → `AiChat` | features; S; not bottom nav |
| `groundzy-wizard` | `@/app/drawers/groundzy-wizard` → `GroundzyWizard` | optional **`tab`** prop; **`hideFromNav`**; **`showHeaderNav` true** |
| `tutorial` | `@/app/drawers/tutorial` → `Tutorial` | informational; **both nav false** (dashboard card) |
| `help` | `@/app/drawers/help` → `Help` | informational; both nav false; **`extendOverBottomNav` false** |
| `contact-us` | `@/app/drawers/contact-us` → `ContactUs` | label **"Inbox"** in registry; S; not bottom nav; order **1.1** (collides with other `1.1` orders) |
| `team-settings` | `@/app/drawers/team-settings` → `TeamSettings` | informational; **Teams tiers**; S; not bottom nav |
| `draw` | `@/app/drawers/draw` → `Draw` | form; `hideFromNav`; **`extendOverBottomNav` true**; full mobile |
| `measure` | `@/app/drawers/measure` → `Measure` | form; `hideFromNav`; **`defaultMobileState` half**; **`extendOverBottomNav` omitted** |

**Duplicate `order` values:** e.g. `trees` and `trees` map items vs `explore`/`ai-identifying-wand`/`contact-us` all use `1.1` or nearby — ordering relies on fractional `order` and category grouping; worth normalizing in refactor.

---

## 3. Implementation files & file sizes

Line counts are **total lines per file** (including blanks). **Largest files under `app/drawers/`** (top 25):

| Lines | File |
|------:|------|
| 1181 | `contact-us.tsx` |
| 1048 | `request-form.tsx` |
| 923 | `view-zone.tsx` |
| 868 | `dashboard.tsx` |
| 843 | `client-form.tsx` |
| 759 | `trees.tsx` |
| 752 | `team-settings.tsx` |
| 748 | `quote-form.tsx` |
| 692 | `invoice-form.tsx` |
| 671 | `job-form.tsx` |
| 659 | `hire-groundzy-pro.tsx` |
| 652 | `profile.tsx` |
| 637 | `ai-chat.tsx` |
| 631 | `view-request.tsx` |
| 626 | `weather-impact.tsx` |
| 563 | `view-tree/components/TreeOverviewTab.tsx` |
| 558 | `view-tree/index.tsx` |
| 523 | `view-property.tsx` |
| 514 | `explore/PostCard.tsx` |
| 507 | `view-tree/components/TreePassport.tsx` |
| 478 | `view-tree/ViewTreeContent.tsx` |
| 477 | `restricted-tree/PublicCommunityEcologyTab.tsx` |
| 451 | `profile/ProfilePlanBillingCard.tsx` |
| 435 | `upgrade-to-teams.tsx` |
| 430 | `property-form.tsx` |

**`useI18n` usage:** ~39 files under `app/drawers` reference `useI18n` — many drawers still **do not** (grep hits are a subset of all drawer files). Expect **mixed i18n** across the tree.

**`useDrawer()` usage:** ~40 files — drawers that navigate or read params typically use it; simple/static drawers may not.

**`components/drawer-layout` (`DrawerHeader`, `DrawerSection`):** Used **sparsely** (e.g. `dashboard.tsx`, `weather-impact.tsx`, some view-* and upgrade drawers). Many drawers roll their own layout.

---

## 4. Per-drawer clusters (deep pass)

### 4.1 Core map & discovery

| Drawer | Purpose / entry | Data / notes | Layout / i18n |
|--------|------------------|--------------|----------------|
| **dashboard** | Home hub; default on `/` | Cards, links to weather/tutorial/help | Large file; `DrawerHeader`/`DrawerSection`; i18n partial |
| **weather** | Weather context | Dashboard card only | Drawer layout components |
| **trees** | Map items list | Map-linked | List patterns |
| **explore** | Social/explore | Tabs; `PostCard` heavy | **Shell** `explore/index.tsx` small; **children** large (`PostCard.tsx` 514 lines) |
| **search** | Search UI | Map/search entry | `useDrawer` |

### 4.2 Map tools (hidden from nav)

| Drawer | Notes |
|--------|--------|
| **tree-add**, **multiple-add**, **search**, **view-tree**, **entry-form**, **restricted-tree**, **edit-tree**, **view-zone**, **draw**, **measure** | Props-driven IDs; `requiredProps` where typed. **view-tree** splits into `ViewTreeDrawerLayout`, `ViewTreeContent`, many tab components (several **300–560+** lines). **view-zone** is **1326 lines** — top refactor target. **draw** has special URL/mobile behavior in `work-area-content`. |

### 4.3 CRM (Pro+)

| Drawer | Notes |
|--------|--------|
| **clients-properties**, **clients**, **properties** | **Same component** (`clients-properties.tsx`); different ids for routing/tabs — document internal mode switching. |
| **view-client**, **add-client**, **edit-client** | **client-form.tsx** 773 lines for add/edit |
| **view-property**, **add-property**, **edit-property** | **property-form.tsx** 360 lines |

### 4.4 Teams workflow (quotes / jobs / requests / invoices)

- **List drawers:** `requests`, `quotes`, `jobs`, `invoices` — sidebar yes, **bottom nav no** (see registry).
- **Forms:** shared patterns; **request-form** is **1234 lines**; quote/job/invoice forms **600–750** lines.
- **Dirty workflow:** only **teams workflow** add/edit drawers are in `WORKFLOW_DRAWERS` — **client/property** forms are **not** (by design or oversight — confirm product intent).

### 4.5 Account & upgrades

| Drawer | Notes |
|--------|--------|
| **profile** | **1900 lines** — largest drawer file; refactor priority **P0** for size |
| **public-profile**, **my-photos** | `my-photos` hidden from both navs |
| **hire-groundzy-pro**, **upgrade-*** | Upgrade category + `getUpgradeGlowClass`; **upgrade-to-teams** uses **form** archetype |

### 4.6 AI / wizard

| Drawer | Notes |
|--------|--------|
| **ai-chat** | **1081 lines** |
| **ai-identifying-wand** | Thin wrapper (56 lines) — logic may live elsewhere |
| **groundzy-wizard** | `hideFromNav`; mobile bottom-nav entry via `getDashboardNavItemsForMobile` substitution |

### 4.7 Messaging / help

| Drawer | Notes |
|--------|--------|
| **contact-us** | **1533 lines** — "Inbox" label; order collision with other `1.1` |
| **help**, **tutorial** | Hidden from nav; entry from dashboard |

### 4.8 Teams-only

| **team-settings** | 751 lines |

---

## Deliverable A — Master table

**Legend — Nav:** `S` = sidebar, `B` = bottom nav, `—` = hidden from nav. **Tiers:** `H+` = `HOME_PLUS_PRO_AND_TEAMS`, `P+` = Pro…Enterprise, `T` = Teams-only, `H/P` = Home+Plus, etc.

| Drawer id | File(s) (primary) | Archetype | Lines (main) | Nav | Tier rules | Top 3 issues | P |
|-----------|-------------------|-----------|-------------:|-----|------------|--------------|---|
| dashboard | `dashboard.tsx` | informational | 868 | S+B | H+ | Size; mixed layout primitives; i18n gaps | P1 |
| weather | `weather-impact.tsx` | informational | 626 | — | H+ | Card-only; medium size | P2 |
| trees | `trees.tsx` | list | 759 | S+B | H+ | Size; list UX | P1 |
| tree-add | `tree-add.tsx` | form | 138 | — | all | Small | P3 |
| multiple-add | `multiple-add.tsx` | form | 403 | — | all | `extendOverBottomNav` vs map UX | P2 |
| search | `search.tsx` | informational | 275 | — | all | | P2 |
| view-tree | `view-tree/index.tsx` + subtree | detail | 558 (+ large children) | — | all | Many huge tab files; prop `treeId` | P0 |
| entry-form | `entry-form/index.tsx` | form | 237 | — | all | `withRetry` only here | P2 |
| restricted-tree | `restricted-tree.tsx` + `PublicCommunityContent.tsx` | detail | 317 + 1034 | — | all | PublicCommunityContent size | P1 |
| edit-tree | `edit-tree.tsx` | form | 54 | — | all | Thin | P3 |
| view-zone | `view-zone.tsx` | detail | **1326** | — | all | **Extreme file size** | **P0** |
| clients-properties | `clients-properties.tsx` | list | 408 | S+B | P+ | Same file as clients/properties | P1 |
| clients | same | list | 408 | — | P+ | Duplicate id pattern | P1 |
| view-client | `view-client.tsx` | detail | 367 | — | P+ | | P2 |
| add-client / edit-client | `client-form.tsx` | form | 773 | — | P+ | Large form | P1 |
| properties | same as clients-properties | list | 408 | — | P+ | | P1 |
| view-property | `view-property.tsx` | detail | 523 | — | P+ | | P1 |
| add/edit-property | `property-form.tsx` | form | 360 | — | P+ | | P2 |
| requests | `requests.tsx` | list | 295 | S | T | Teams-only | P2 |
| view-request | `view-request.tsx` | detail | 631 | — | T | | P1 |
| add/edit-request | `request-form.tsx` | form | **1234** | — | T | **Very large** | **P0** |
| quotes | `quotes.tsx` | list | 209 | S | T | | P2 |
| view-quote | `view-quote.tsx` | detail | 340 | — | T | | P2 |
| add/edit-quote | `quote-form.tsx` | form | 748 | — | T | Large | P1 |
| jobs | `jobs.tsx` | list | 194 | S | T | | P2 |
| view-job | `view-job.tsx` | detail | 230 | — | T | | P2 |
| add/edit-job | `job-form.tsx` | form | 671 | — | T | | P1 |
| invoices | `invoices.tsx` | list | 190 | S | T | | P2 |
| view-invoice | `view-invoice.tsx` | detail | 205 | — | T | | P2 |
| add/edit-invoice | `invoice-form.tsx` | form | 692 | — | T | | P1 |
| profile | `profile.tsx` | informational | **1900** | S | H+ | **Largest drawer** | **P0** |
| public-profile | `public-profile.tsx` | detail | 154 | — | all | | P2 |
| my-photos | `my-photos.tsx` | informational | 363 | — | all | Hidden nav | P2 |
| hire-groundzy-pro | `hire-groundzy-pro.tsx` | informational | 656 | S | H/P | | P1 |
| upgrade-to-plus | `upgrade-to-plus.tsx` | informational | 128 | S | Home | | P3 |
| upgrade-to-pro | `upgrade-to-pro.tsx` | informational | 175 | S | H+P | | P3 |
| upgrade-to-teams | `upgrade-to-teams.tsx` | form | 429 | S | H+P+Pro | form archetype | P2 |
| explore | `explore/index.tsx` | informational | 92 (shell) | S+B | H+ | Fat `PostCard` / tabs | P1 |
| ai-identifying-wand | `ai-identifying-wand.tsx` | informational | 56 | S | H+ | **showHeaderNav vs comment** | P2 |
| ai-chat | `ai-chat.tsx` | informational | 1081 | S | H+ | Size | P0 |
| groundzy-wizard | `groundzy-wizard.tsx` | informational | 128 | — | H+ | Mobile-only nav | P2 |
| tutorial | `tutorial.tsx` | informational | 287 | — | H+ | | P2 |
| help | `help.tsx` | informational | 472 | — | H+ | | P2 |
| contact-us | `contact-us.tsx` | informational | **1533** | S | H+ | **Very large**; label "Inbox"; order collision | P0 |
| team-settings | `team-settings.tsx` | informational | 751 | S | Teams | | P1 |
| draw | `draw.tsx` | form | 340 | — | all | Map special-case | P2 |
| measure | `measure.tsx` | form | 151 | — | all | half sheet default | P3 |

---

## Deliverable B — Inconsistency list (by theme)

### Layout / structure

- **`DrawerHeader` / `DrawerSection`** (`components/drawer-layout/`) are **not** used consistently; many drawers use ad hoc `div` + `className` patterns.
- **Archetype** in metadata (`informational` | `form` | `list` | `detail`) is **not enforced** in code — no runtime check that UI matches archetype.
- **Same physical drawer** (`clients-properties.tsx`) backs **three ids** (`clients-properties`, `clients`, `properties`) — behavior must stay in sync when refactoring.

### i18n

- **`useI18n` / `t()`** is present in many but **not all** drawer files; expect **hardcoded English** in large files (e.g. profile, contact-us, help) unless verified.
- Registry labels (`label: "Inbox"` for `contact-us`) are **English** — may need i18n for nav if product requires translations.

### Types / registry

- **Generic props** (`registerDrawer<{...}>`) are declared for **some** drawers; **URL params are strings** — validation is implicit (merge in `DrawerWrapper`).
- **`showHeaderNav`** comment on **`ai-identifying-wand`** contradicts **value `true`** (`lib/drawers.ts`).

### Nav metadata

- **Duplicate `order` values** (e.g. multiple `1.1`) — stable today but fragile for sorting/grouping.
- **`contact-us`** `order: 1.1` matches **trees**/**explore**/**ai-identifying-wand** space — odd grouping in More/dashboard sections.
- **`groundzy-wizard`**: comment says "mobile bottom nav only" but **`hideFromNav`** excludes it from **getBottomNavDrawerMetadata** — actual entry is **mobile dashboard** via `getDashboardNavItemsForMobile` (see `drawer-registry.ts`).

### Styling / UX

- **`extendOverBottomNav`** varies **form** drawers (many `true` for add/edit) vs **map** drawers (`tree-add` false, `multiple-add` true`) — verify **consistent thumb reach** and **keyboard** behavior on mobile.
- **Workflow dirty** only for **teams** CRM forms — **client/property** forms not in `WORKFLOW_DRAWERS` — confirm intentional.

### Accessibility (spot)

- **No systematic audit** performed; `DrawerWrapper`/`work-area` should own **title** alignment with drawer label — verify **one** `h1` / focus order per open drawer.

---

## Deliverable C — Refactor backlog (ordered)

| # | Goal | Affected drawers / files | Shared abstraction | Risk | Split-only vs behavior |
|---|------|--------------------------|--------------------|------|-------------------------|
| 1 | Split **view-zone** | `view-zone.tsx` | Zone sections + hooks | Low if pure extract | Split-only |
| 2 | Split **profile** | `profile.tsx` | Section components, profile hooks | Medium | Split-only |
| 3 | Split **contact-us** / inbox | `contact-us.tsx` | Thread list, composer, message row | Medium | Split-only |
| 4 | Split **request-form** | `request-form.tsx` | Field groups, validation, steps | Medium | Split-only |
| 5 | **view-tree** tab extraction | `view-tree/**` | Tree tab scaffolds already partial — continue | Medium | Split-only |
| 6 | **Explore** fat components | `explore/PostCard.tsx`, tabs | Card primitives, skeletons | Low | Split-only |
| 7 | **ai-chat** modularization | `ai-chat.tsx` | Chat layout, message list, input | Medium | Split-only |
| 8 | Introduce **DrawerShell** (optional) | All drawers | Standard scroll region + optional footer slot | **High** (touches many) | Behavior if focus/scroll changes |
| 9 | **i18n sweep** | Drawers missing `useI18n` | Key conventions in `lib/i18n/messages.ts` | Low (copy) | Behavior if strings change |
| 10 | Normalize **order** / **category** | `lib/drawers.ts` | Single source of truth for nav groups | Low | **Possible** nav order change |
| 11 | Fix **showHeaderNav** comment or value | `ai-identifying-wand` | — | Low | **Behavior** if toggled |
| 12 | Align **WORKFLOW_DRAWERS** with product | client/property forms? | Extend store set | Medium | **Behavior** |

---

## Deliverable D — Golden reference & worst offenders

### Golden reference (target patterns)

1. **`dashboard.tsx`** — Uses `DrawerHeader` / `DrawerSection`, cards, clear sections; **large** but **structured** (good reference for **sectioning**, not file size).
2. **Small map drawers** — **`edit-tree.tsx`** (54 lines) / **`ai-identifying-wand.tsx`** (56 lines) — Thin shells that delegate to hooks/components (pattern for **minimal** drawer files).

### Worst offenders (why)

1. **`profile.tsx` (~1900 lines)** — Single-file concentration; hard to test and review.
2. **`view-zone.tsx` (~1326 lines)** — Detail drawer with no split; highest **geographic** complexity likely.
3. **`contact-us.tsx` (~1533 lines)** — Inbox + messaging in one file.
4. **`request-form.tsx` (~1234 lines)** — Largest **form**; teams workflow critical path.

---

## Runtime verification checklist (ambiguous items)

- **Mobile:** `extendOverBottomNav` for each **add/edit** form vs keyboard and safe area.
- **Tier:** Open Teams-only URLs as **Pro** user — confirm redirect in `work-area-content` (`canUserAccessDrawer`).
- **Wizard:** `ai-identifying-wand` header back/forward — **should** they show? Compare to product; fix `showHeaderNav` or comment.
- **`draw` drawer:** Narrow viewport + no polygon → sheet `closed` (`work-area-content.tsx`).

---

## Visual system inventory (appendix)

Per-drawer shell, scroll, and card-pattern classification (for styling refactors): [`drawer-visual-inventory.md`](drawer-visual-inventory.md).

---

*End of audit — all counts and registry rows reflect `lib/drawers.ts` and `app/drawers` as of the audit date.*

---

## Implementation follow-up (post–plan v3)

- **DrawerShell / DrawerScrollArea** — See [components/drawer-layout/drawer-shell.tsx](../components/drawer-layout/drawer-shell.tsx), [drawer-scroll-area.tsx](../components/drawer-layout/drawer-scroll-area.tsx). `DrawerBody` re-exports `DrawerScrollArea` as a backward-compatible alias.
- **PR checklist** — [DRAWER_PR_CHECKLIST.md](./DRAWER_PR_CHECKLIST.md)
- **Navigation spec** — [drawer-navigation.md](./drawer-navigation.md)
- **Remaining megafile splits** — Profile, contact-us, view-zone, request-form, ai-chat, view-tree tabs, and large CRM content remain candidates for incremental extraction (see plan phases).
