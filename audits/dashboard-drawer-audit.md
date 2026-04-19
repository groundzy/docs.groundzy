# Dashboard Drawer — Complete Code Audit

> Source of truth: code only. Compiled from `app/drawers/dashboard.tsx`, `app/drawers/dashboard/*`, `lib/dashboard-surface-access.ts`, `lib/utils/dashboard-ai-tip.ts`, `lib/utils/tier-utils.ts`, `lib/participant-work-surface-access.ts`, `hooks/useTeamsOnlyAccess.ts`, `components/weather/WeatherCard.tsx`, `lib/drawers.ts`, `lib/drawer-registry.ts`.
>
> Scope: every visible block in the Dashboard drawer, what data it draws from, and exactly which tiers / roles see (or omit) each piece. Where the code disagrees with its own comments, that is called out explicitly.

---

## 1. Drawer registration & visibility

Registered in `lib/drawers.ts`:

```81:92:lib/drawers.ts
registerDrawer("dashboard", {
  label: "Dashboard",
  icon: LayoutDashboard,
  order: 1,
  archetype: "informational",
  role: "primary",
  visibleForTiers: HOME_PLUS_PRO_AND_TEAMS,
  visibleInSidebar: true,
  visibleInBottomNav: true,
  bottomNavPriority: 1, // High priority for bottomNav
  defaultMobileState: "full",
}, withRetry(() => import("@/app/drawers/dashboard").then(m => ({ default: m.Dashboard }))));
```

- `visibleForTiers: HOME_PLUS_PRO_AND_TEAMS` — every supported tier (Home, Plus, Pro, Small/Mid/Large Team, Enterprise).
- Bottom-nav priority 1 → first slot on mobile.

### Defense-in-depth content gate

```18:20:lib/dashboard-surface-access.ts
export function canAccessDashboardSurface(_args: DashboardSurfaceArgs): boolean {
  return true;
}
```

The dashboard component calls `canAccessDashboardSurface(...)` and returns `null` when it is false (`app/drawers/dashboard.tsx` lines 119–123, 389–391). **In current code that gate never fires** — the function is a stub that always returns `true`. The comment block above the call says “Dashboard is hidden for Teams non-owner/admin members”, but this is *aspirational only* — the gate has been disabled. As written, every authenticated user receives the same dashboard surface.

> Audit note: comment vs. code drift. Either re-introduce the role gate inside `canAccessDashboardSurface` or update the comment.

---

## 2. Top-level component data inputs

`app/drawers/dashboard.tsx` (Dashboard component, lines 107–706) reads:

| Input | Hook / source | Used for |
|---|---|---|
| `user`, `userDoc`, `organization` | `useAuth()` | identity, name, tier resolution, org name |
| `organizationId` | `organization.id` ⇢ `userDoc.organizationId` ⇢ `user.uid` | scoping queries |
| `trees` | `useTrees(organizationId)` | health, species, “not assessed”, trends, upcoming |
| `speciesCatalog` | `useSpeciesCatalog()` | localized species names for upcoming items |
| `hasTeamsAccess` | `useTeamsOnlyAccess()` ⇢ capability `workflow.pipeline` | gates Work-this-week + Requests/Jobs quick actions |
| `jobs` | `useJobs(organizationId)` | jobs today (weather card), jobs this week (Teams) |
| `prefs` | `userDoc.preferences` | feature flag `dashboardWorkMerge` |
| `scheduledWorkItems` | `useScheduledWorkItemsForDashboard(orgId, dashboardWorkMerge)` | merged into `upcoming` when flag on |
| `upcoming` | `getDashboardUpcoming(...)` (+ optional merge with work items) | greeting next-due, AI tip, tree side of Upcoming card |
| `participantItems` | `useParticipantWorkflow()` (gated by `canAccessParticipantWorkSurface`) | participant rows in Upcoming card |
| `participantClients` | `useClients(...)` (only when `canSeeParticipantWork`) | client/contact display name on participant rows |
| `participantToday` | filter via `itemMatchesScheduleChip(row, "today")` | participant rows narrowed to today (UTC-day chip semantics) |
| `upcomingTreeToday` | `filterDashboardUpcomingToToday(upcoming, ...)` | tree rows narrowed to today (local-day) |
| `treeUsage`, `aiUsage` | `useTreeUsage()`, `useAiUsage()` | Plan card limits + add-tree gate |
| `notificationsUnread`, `unreadByConversation` | `useUnreadNotifications`, `useUnreadMessages` | inbox badge on Upcoming header |
| `pendingAccessRequests` | `usePendingAccessRequestsForOwner(organizationId)` | mobile sections inbox dot |
| `mapCenter`, `weatherAnchor`, `weatherCoords` | `useMapStore`, `useWeatherMapAnchor` | weather query coords |
| `isFahrenheit`, `isImperial` | `useUserPreferences()` | weather units |
| `weatherData` | `useWeatherContext(lat, lng, {unitGroup, lang})` | WeatherCard body |
| `mapboxPlaceLabel` | `useMapboxPlaceLabel(...)` | location string in WeatherCard |
| `effectiveTier` | `getEffectiveSubscriptionTier(userDoc)` | every tier branch below |
| `weatherCardTier` | `getWeatherCardTier(effectiveTier)` | controls WeatherCard variant |
| `weatherRecommendations` | `computeRecommendations(weatherData, trees, jobs, …)` — only when `weatherCardTier ∈ {pro, team}` | Pro/Team weather advisories + AI tip pri 2/7 |
| `aiTip` | `getPrioritizedAiTip(...)` | Groundzy AI Wizard card |

### Tier resolution rules (used by every section below)

`lib/utils/tier-utils.ts`:

- `getEffectiveSubscriptionTier(userDoc)` returns `null | "Home" | "Plus" | "Pro" | "Small Team" | "Mid Team" | "Large Team" | "Enterprise"`. **Plus/Pro/Teams downgrade to `"Home"` whenever `subscription.status` is not `active|trialing`.**
- `getWeatherCardTier`:
  - `null` → `"home"`
  - `"Home"` → `"home"`
  - `"Plus"` → falls through ⇒ `"home"` (Plus does **not** unlock the pro/team weather UI)
  - `"Pro"` → `"pro"`
  - any Team / Enterprise → `"team"`
- `useTeamsOnlyAccess().hasAccess` ≡ capability `workflow.pipeline` (Pro + all Team tiers).
- `canAccessParticipantWorkSurface`:
  - Pro → always true
  - Team tier → only if user is in a real team org **and** their org preset is `"fieldworker"`
  - Otherwise → false

These three predicates produce **four functional roles** the dashboard distinguishes (independent of tier):

| Role flag | Value source | What it unlocks on the dashboard |
|---|---|---|
| `canSeeDashboard` | `canAccessDashboardSurface` (currently always `true`) | the entire drawer |
| `weatherCardTier` (`home` / `pro` / `team`) | `getWeatherCardTier(effectiveTier)` | WeatherCard variant + recommendations + AI tip 2/7 |
| `hasTeamsAccess` | capability `workflow.pipeline` | Work this week + Requests/Jobs quick actions |
| `canSeeParticipantWork` | `canAccessParticipantWorkSurface` | participant rows in Upcoming + “View all work” footer |

There is no separate “admin/owner” gate inside the Dashboard component; org-role logic only enters indirectly through `canAccessParticipantWorkSurface` (fieldworker preset on Team tiers) and through `useTeamsOnlyAccess`.

---

## 3. Render order (top → bottom)

Implemented in the `return` block of `app/drawers/dashboard.tsx` (lines 393–705). Top-level wrapper is `<DrawerShell><DrawerBody className="space-y-6">…</DrawerBody></DrawerShell>`.

1. **Weather + greeting card** — `WeatherCard` if `hasWeatherLocation`, else `greetingBlock` + “center map for weather” prompt.
2. **Upcoming services & inspections** — `DashboardUpcomingList` inside a `DrawerSection`, with an inbox bell as `headerAction`.
3. **Tutorial card** — `DashboardTutorialCard` (self-hides when nothing to show).
4. **Groundzy AI Wizard card** — `DashboardAiWizardCard` with `getPrioritizedAiTip(...)`.
5. **Your Trees** — `DashboardYourTreesSection`.
6. **Work this week** — Teams-only `DrawerSection` with up to 5 jobs in next 7 days.
7. **Plan & account** — clickable `DrawerSection` summarizing tier + limits + org name.
8. **Quick actions** — grid of buttons.
9. **Mobile-only sections** — `DashboardMobileDashboardSections` (Upgrade + “All sections”).
10. **Footer** — Visual Crossing attribution link.

---

## 4. Card-by-card audit

### 4.1 Weather card / greeting (block 1)

Code path:

```397:422:app/drawers/dashboard.tsx
{hasWeatherLocation ? (
  <WeatherCard ... tier={weatherCardTier} ... greeting={greeting} userName={firstName} nextDue={...} />
) : (
  <>
    {greetingBlock}
    <p>{t("dashboard.centerMapWeather")}</p>
  </>
)}
```

- `hasWeatherLocation` requires `weatherCoords` from `useWeatherMapAnchor(mapCenter)`. If the user hasn’t centered the map yet, the greeting block appears alone.
- `greetingBlock` is a `DrawerSection` with: time-of-day greeting (`getGreetingKey()`), first name, and either a yellow “Your *tree* has *label* *(today / tomorrow / on …)*” warning button (when `upcoming[0]` exists) or the `dashboard.nothingDueSoon` line.

`WeatherCard` content varies by `tier` (`home | pro | team`), `lib/components/weather/WeatherCard.tsx`:

| Element | home | pro | team |
|---|---|---|---|
| Header (greeting / userName / nextDue) | ✓ | ✓ | ✓ |
| Location row + impact pill | ✓ | ✓ | ✓ |
| **Frost pill** (next-N-days) | ✓ (only on `home`, line 219) | — | — |
| Big temperature + conditions | ✓ | ✓ | ✓ |
| “Feels like X°” suffix | — | ✓ (when `feelslike !== temp`) | ✓ (when `feelslike !== temp`) |
| Wind / precip% / precip-amount column | ✓ | ✓ | ✓ |
| Gusts + humidity micro-row | — | ✓ | ✓ |
| 3-day forecast strip (`forecastDays=3`) | ✓ | ✓ | ✓ |
| “N jobs today” pill (`jobsTodayCount`) | — | — | ✓ (only when count > 0) |
| Locked “Job Impact (Team only)” strip | — | ✓ | — |
| First weather advisory (`firstAdvisory.message`) | — | ✓ (when present) | ✓ (when present) |
| “Severe weather → upgrade to Team” hint | — | ✓ (when `level === "high" || advisory.priority === "high"` **and** `onUpgrade` is wired) | — |
| Aria label | `t("weather.ariaFull")` | `t("weather.ariaPro")` | `t("weather.ariaTeam")` |
| “View full forecast →” CTA | ✓ | ✓ | ✓ |

`weatherRecommendations` itself is gated:

```305:308:app/drawers/dashboard.tsx
const weatherRecommendations =
  weatherData && (weatherCardTier === "pro" || weatherCardTier === "team")
    ? computeRecommendations(weatherData, trees, jobs, { unitGroup })
    : [];
```

So `home`/`Plus` users (since Plus → `home` for weather) never receive `recommendations`, and consequently never get advisory text in the card or in the AI tip pri 2/7 (see §4.4).

`onUpgrade` is wired to `navigate("settings", { tab: "personal" })` on every tier (line 407) — but the Pro-only “upgrade to Team” hint only renders when the conditions in the table above also hold.

### 4.2 Upcoming services & inspections (block 2)

Section header (`DrawerSection`, `compact`, `icon={CalendarClock}`, `accent="icon"`):

- Title = `dashboard.upcoming.title`, subtitle = `dashboard.upcoming.description`.
- `headerAction` is a circular **inbox bell** linking to `contact-us?tab=activity`. The bell flips to a `destructive`-tinted style and shows a numeric badge (`99+` cap) when `inboxUnreadCount = notificationsUnread + messagesUnread > 0`.

Body = `<DashboardUpcomingList … />` (`app/drawers/dashboard/DashboardUpcomingList.tsx`):

- Loading state (`isLoading` is true while `treesLoading` OR while merged-work-items loading is in-flight OR while participant data loads for fieldworkers): shows `dashboard.loading`.
- Empty state (no participant + no tree rows for today): `dashboard.nothingScheduled`.
- Otherwise renders a single `<ul>` with two row archetypes, **participant rows first**:

#### Participant rows (only when `canSeeParticipantWork === true` AND `participantToday.length > 0`)

For each `ParticipantWorkflowListItem` not of type `"invoice"`:

- Icon well: workflow pipeline icon (`WORKFLOW_PIPELINE_ICONS[participantKindWorkflowId[row.type]]`), styled with `workflowStepIconVars(workflowId)` — uses the global `--workflow-*` translucent palette per `.cursor/rules/workflow-step-colors.mdc`.
- Title: `row.doc.title || #<number>`.
- Sub-line: `kindLabel · statusLabel · clientLine` (joined with ` · `, falsy parts filtered).
  - `kindLabel`: `participantWork.kindRequest|Quote|Job|Invoice`.
  - `statusLabel`: for jobs uses fieldworker-specific i18n key (`JOB_STATUS_I18N_KEY_FIELDWORKER`), else `formatWorkflowStatusLabel(status)`.
  - `clientLine`: `getClientDisplayName(client) [– getContactDisplayName(contact)]`, falling back to `clientDisplayNameSnapshot`.
- Click → `openParticipantItem(row)` → `navigate("view-request"|"view-quote"|"view-job", {...Id})`. Invoices are filtered out at the data layer above.
- `data-dashboard-upcoming-row="participant-<type>"` for testing.

#### Tree rows (`upcomingTreeToday` filtered from `getDashboardUpcoming(trees, locale, speciesCatalog)`)

- Icon well uses the **tree** translucent token system (not the workflow palette):

```67:80:app/drawers/dashboard/DashboardUpcomingList.tsx
const treeTypeConfig = {
  service:    { icon: CalendarCheck,  bg: --tree-service-tile/0.35,    text: --tree-service,    hoverBg: 0.5 },
  inspection: { icon: Stethoscope,    bg: --tree-inspection-tile/0.35, text: --tree-inspection, hoverBg: 0.5 },
};
```

- Title: `item.treeLabel`.
- Sub-line: `getTypeLabel(item, t)` (translates `dashboard.upcoming.serviceType.<key>` / `inspectionType.<key>`, with raw `item.label` fallback) + `formatDate(item.date, locale)`.
- Click → `onOpenTree(item.treeId)` → `navigate("view-tree", { treeId })`.
- `data-dashboard-upcoming-row="tree-<type>"`.

Footer:

- “View all work →” button — only renders when `showViewAllWork === true`, i.e. when `canSeeParticipantWork` is true (Pro tier OR Team-tier fieldworker). Goes to `participant-work` drawer.

#### Two different “today” semantics

```168:183:app/drawers/dashboard.tsx
// Participant rows use chip semantics (UTC-day) so the dashboard always agrees with
// the Worker Agenda drawer; tree rows use local-day to match getDashboardUpcoming’s
// wall-clock filter.
```

> Audit note: the two halves of the same list intentionally use different day boundaries (UTC vs local). Documented in the source; worth keeping in mind when triaging “why is X showing on a different day on dashboard vs. agenda”.

### 4.3 Tutorial card (block 3)

`app/drawers/dashboard/DashboardTutorialCard.tsx`:

- Resolves `userTier = getEffectiveSubscriptionTier(userDoc)` and `tutorials = getTutorialsForTier(userTier)`.
- Self-hides (`return null`) when:
  - `useTutorialState().showTutorials === false`, OR
  - `allTutorialsCompleted === true`, OR
  - `tutorials.length === 0`.
- Otherwise picks `nextTutorial = first incomplete OR tutorials[0]`, looks up `i18n` keys via `tutorialTitleKey(id)` / `tutorialShortDescriptionKey(id)` with raw `title` / `shortDescription` as fallback.
- Footer row: `dashboard.tutorialCard.progress` (`current/total`) + CTA — `dashboard.tutorialCard.cta` if next is already completed (re-watch), otherwise `dashboard.tutorialCard.ctaContinue`.
- Click navigates to `tutorial` drawer.

### 4.4 Groundzy AI Wizard card (block 4)

`app/drawers/dashboard/DashboardAiWizardCard.tsx`:

- Always rendered. `isLoading={treesLoading}` shows a 4-bar shimmer skeleton.
- Uses `--stormy-teal` background, `--tea-green` accents, decorative `Sparkles` watermark on the right.
- Click → `navigate("groundzy-wizard", { tab: "chat" })`.
- Renders `tip.title?` (`titleKey` if present) above `tip.body` (`bodyKey`/`bodyParams` if present), then a `dashboard.askMore` CTA with chevron.

`tip` comes from `getPrioritizedAiTip(...)` (`lib/utils/dashboard-ai-tip.ts`). **Priority order — first match wins:**

| # | Trigger | Urgency | i18n / message |
|---|---|---|---|
| 1 | `needsAttentionCount > 0` | critical | `aiTip.needsAttentionOne` / `aiTip.needsAttentionOther({count})` |
| 2 | `weatherRecommendations` has `priority === "high"` | critical | raw `rec.title` + `rec.message` (only ever populated for `pro`/`team` weather tier) |
| 3 | `jobsTodayCount > 0` | high | `aiTip.jobsTodayOne` / `aiTip.jobsTodayOther({count})` |
| 4 | `upcoming[0]` present | high | `aiTip.upcoming({treeLabel, label, relativeDate})` (today / tomorrow / `onDate`) |
| 5 | `trendSummary.declining > 0` | high | `aiTip.decliningOne` / `aiTip.decliningOther({count})` |
| 6 | `notAssessedIn12Months > 0` | high | `aiTip.notAssessedOne` / `aiTip.notAssessedOther({count})` |
| 7 | any other `weatherRecommendations[0]` | normal | raw `rec.title` + `rec.message` (Pro/Team only) |
| 8 | fallback | normal | `aiTip.seasonal.<month0..11>` |

Implication by tier: Home and Plus dashboards never reach pri 2 or 7 (no `weatherRecommendations`); they only ever see the seven other branches.

### 4.5 Your Trees (block 5)

`app/drawers/dashboard/DashboardYourTreesSection.tsx`. Section icon `TreePine`.

- **Loading**: `dashboard.loading`.
- **Empty (`treesLength === 0`)**: copy `dashboard.addFirstTree` + a primary “+ Add tree” button → `onAddTree()` → `handleAddTreeClick`. The handler enforces the tree-usage limit:

```324:332:app/drawers/dashboard.tsx
if (!treesLimit.isUnlimited && treesLimit.used >= treesLimit.limit) {
  toast.error(t("limits.treeLimitReached", { limit: treesLimit.limit }), {
    action: { label: t("limits.upgradeToPlus"), onClick: startPlusCheckout },
  });
  return;
}
navigate("tree-add");
```

- **Non-empty body** is composed of four blocks:
  1. **Counts row**: `<bold>treesLength</bold> tree(s) · <bold>speciesCount</bold> species`. If `notAssessedIn12Months > 0`, append a `--tea-green` link `dashboard.notAssessedIn12Months({count})` → `navigate("trees")`.
  2. **Health status grid** (2 cols mobile, 3 desktop): only renders the buckets with `count > 0`. Color mapping:
     - `excellent`, `good`, `fair`, `dead` → muted-foreground
     - `poor` → `text-warning`
     - `critical` → `text-destructive`
  3. **Health trend block** — only when `trendSummary.treesWithInspections > 0`. Header is `dashboard.basedOnAssessments({period: trendSummary.periodLabel})` if `periodLabel` exists, else `dashboard.healthTrends`. Body = `<DashboardTrendSummary>` — three compact cards with `improving` (success), `stable` (foreground), `declining` (warning). When `treesWithInspections === 0`, `DashboardTrendSummary` itself shows `dashboard.addAssessmentsForTrends`.
  4. **Bottom CTA** (above a `border-t`):
     - All healthy (`healthyCount === treesLength && needsAttentionCount === 0`): copy `dashboard.allTreesGoodShape({count, treeWord})`. If `showHirePro`, append a “Schedule a Pro” inline link.
     - Else if `needsAttentionCount > 0`: copy `dashboard.considerAssessment({count, treeWord})`, then a flex row of buttons:
       - `Hire a Pro` button → `navigate("hire-groundzy-pro")` (only when `showHirePro`)
       - `Filter by health` link → `navigate("trees")`
     - Else (mixed but nothing critical): no CTA block.

`showHirePro = effectiveTier === "Home" || effectiveTier === "Plus"` (line 316). Pro / Team tiers therefore see **no** “Hire a Pro” affordances anywhere on the dashboard.

### 4.6 Work this week (block 6)

```516:558:app/drawers/dashboard.tsx
{hasTeamsAccess && jobsThisWeek.length > 0 && (
  <DrawerSection title="Work this week" icon={Shovel} ...>
    {/* up to 5 jobs in next 7 days, sorted by source order */}
  </DrawerSection>
)}
```

- Visibility: requires `useTeamsOnlyAccess().hasAccess` — i.e. the `workflow.pipeline` capability. In practice that is **Pro + every Team tier**. Home / Plus never see it.
- Data: `jobs.filter(...)` where `schedule.startDate` falls in `[now, now + 7d]` and `status !== "archived"`, then `.slice(0, 5)`.
- Each row: title (truncated) + short date (`{month: "short", day: "numeric"}`) → `navigate("view-job", { jobId })`.
- Footer link: `dashboard.viewAllJobs` → `navigate("jobs")`.
- Hidden when `jobsThisWeek.length === 0` even for Teams users.

### 4.7 Plan & account (block 7)

Visible iff `planTier || orgName`:

```309:314:app/drawers/dashboard.tsx
const planTier = effectiveTier
  ? String(effectiveTier).toLowerCase().includes("plan")
    ? effectiveTier
    : `${effectiveTier} Plan`
  : null;
const orgName = organization?.name ?? userDoc.organizationName;
```

- Whole card is one button → `navigate("settings", { tab: "personal" })`.
- Right-aligned **tier badge** in the section title:

| `effectiveTier` value | Badge text | Badge classes |
|---|---|---|
| `"Pro"` | `PRO` | `bg-primary/15 text-primary` |
| `"Plus"` | `PLUS` | `bg-[--green-yellow/0.25] text-[--green-yellow]` |
| `"Home"` | `HOME` | `bg-muted text-muted-foreground` |
| Any string containing `"Team"` or `"Enterprise"` | `TEAM` | `bg-primary/15 text-primary` |
| Anything else | `effectiveTier ?? "FREE"` | `bg-muted text-muted-foreground` |

- Body line 1 (only when `planTier` is set): either `dashboard.planUnlimited` (when *all three* of `treesLimit / aiLimits / plantnetLimits` are `isUnlimited`) **or** `dashboard.planFeaturesCompact({trees, ai, ids})` where each value is `"∞"` for unlimited buckets.
- Body line 2 (only when `orgName` is set): `dashboard.organization: <orgName>`.

### 4.8 Quick actions (block 8)

Always rendered. Grid 2 cols mobile / 3 cols desktop. Buttons (in DOM order):

| Order | Label key | Visible when | Action |
|---|---|---|---|
| 1 | `dashboard.trees` | always | `navigate("trees")` |
| 2 | `dashboard.addTree` | always | `handleAddTreeClick` (limit-aware, see §4.5) |
| 3 | `dashboard.profile` | always | `navigate("settings", { tab: "personal" })` |
| 4 | `dashboard.hireAPro` | `showHirePro` (Home or Plus) | `navigate("hire-groundzy-pro")` |
| 5 | `dashboard.requests` | `hasTeamsAccess` (Pro + Teams) | `navigate("requests")` |
| 6 | `dashboard.jobs` | `hasTeamsAccess` (Pro + Teams) | `navigate("jobs")` |
| 7 | `dashboard.settings` | always | `navigate("settings", { tab: "personal" })` |
| 8 | `dashboard.helpFaqs` | always | `navigate("help")` |

So per-tier visible buttons:

| Tier | Visible quick actions |
|---|---|
| Home / Plus (no Pro plan) | Trees, Add tree, Profile, **Hire a Pro**, Settings, Help |
| Pro (solo) | Trees, Add tree, Profile, **Requests**, **Jobs**, Settings, Help |
| Team / Enterprise (any role) | Trees, Add tree, Profile, **Requests**, **Jobs**, Settings, Help |

### 4.9 Mobile-only sections (block 9)

`app/drawers/dashboard/DashboardMobileDashboardSections.tsx`. Both subsections are wrapped in `md:hidden`, so desktop never renders them.

#### a) Upgrade section (`Sparkles` icon)

- Source: `getUpgradeOptionsForTier(userTier)` ⇒ all sidebar drawers with `category === "upgrades"`.
- Per `lib/drawers.ts`:

| Drawer id | `visibleForTiers` | Glow class |
|---|---|---|
| `upgrade-to-plus` | `["Home"]` | `icon-glow-plus` |
| `upgrade-to-pro` | `["Home", "Plus"]` | `icon-glow-pro` |
| `upgrade-to-teams` | `["Home", "Plus", "Pro"]` | `icon-glow-teams` |

So the **mobile Upgrade section** is populated as:

| Tier | Upgrade tiles shown |
|---|---|
| Home | Plus, Pro, Teams |
| Plus | Pro, Teams |
| Pro | Teams |
| Any Team / Enterprise | *(section omitted — `upgradeOptions.length === 0`)* |

Each tile is a `listItemCardTintedClass` button with the registered icon (`--tea-green`), the localized drawer label, and a glow halo around the icon (`icon-glow-plus|pro|teams`).

#### b) “All sections” (`Zap` icon)

- Source: `getDashboardNavItemsForMobile(userTier, sectionLabels, 3, categoryLabels)`.
- Filters out `ai-identifying-wand` and `ai-chat`, force-injects `groundzy-wizard` if missing, then re-buckets by `category` (with order-based fallback) and sorts groups by min `order`.
- Each non-empty group renders its localized `sectionLabel` over a 2-col / 3-col grid of tile buttons identical in shape to the upgrade section.
- The `contact-us` tile gets a small red dot when `inboxUnreadCount > 0 || pendingAccessRequestsCount > 0`, and its `aria-label` becomes `dashboard.openInboxUnread({count: inboxUnreadCount})`.

> Tier impact is implicit: every drawer is filtered through `userTier` inside `getMoreDrawerMetadata` / `getSidebarDrawerMetadata` (see `lib/drawer-registry.ts`), so the “All sections” grid expands as the user’s tier grows.

### 4.10 Footer (block 10)

```692:702:app/drawers/dashboard.tsx
<a href="https://www.visualcrossing.com/" target="_blank" rel="noopener noreferrer">
  {t("dashboard.weatherDataBy")}
  <ExternalLink className="w-3 h-3" />
</a>
```

Always rendered. No tier gating. Required attribution because the dashboard consumes Visual Crossing weather data via `useWeatherContext`.

---

## 5. Tier × role matrix (every visible block)

Rows = tier. `Pro/F` = Pro tier on a fieldworker preset; `Team/F` = Team tier user with `fieldworker` preset; `Team/Other` = Team tier user with any other preset (admin, owner, manager, etc.). Plus column tracks behavior for Plus separately because Plus is treated as `home` by the WeatherCard.

| Block | Home | Plus | Pro (solo) | Pro/F | Team/F | Team/Other |
|---|---|---|---|---|---|---|
| **WeatherCard variant** | `home` | `home` | `pro` | `pro` | `team` | `team` |
| Frost pill | ✓ (home only) | ✓ | — | — | — | — |
| Feels-like / gusts / humidity | — | — | ✓ | ✓ | ✓ | ✓ |
| Jobs-today pill | — | — | — | — | ✓ (when >0) | ✓ (when >0) |
| Locked “Job Impact” strip | — | — | ✓ | ✓ | — | — |
| Weather advisory line | — | — | ✓ (when present) | ✓ | ✓ | ✓ |
| Upcoming card – tree rows | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Upcoming card – participant rows | — | — | ✓ | ✓ | ✓ | — |
| “View all work” footer | — | — | ✓ | ✓ | ✓ | — |
| Inbox bell (badge when unread) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Tutorial card | ✓ (tier-filtered list) | ✓ | ✓ | ✓ | ✓ | ✓ |
| AI Wizard card | ✓ (no pri 2/7) | ✓ (no pri 2/7) | ✓ | ✓ | ✓ | ✓ |
| Your Trees → Hire a Pro button | ✓ | ✓ | — | — | — | — |
| Your Trees → Schedule a Pro inline | ✓ | ✓ | — | — | — | — |
| Your Trees → Filter by health | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Work this week** section | — | — | ✓ (when jobs in next 7d) | ✓ | ✓ | ✓ |
| Plan & account card | ✓ (badge `HOME`) | ✓ (`PLUS`) | ✓ (`PRO`) | ✓ (`PRO`) | ✓ (`TEAM`) | ✓ (`TEAM`) |
| Quick action: Hire a Pro | ✓ | ✓ | — | — | — | — |
| Quick action: Requests | — | — | ✓ | ✓ | ✓ | ✓ |
| Quick action: Jobs | — | — | ✓ | ✓ | ✓ | ✓ |
| Mobile Upgrade section | ✓ (3 tiles) | ✓ (Pro+Teams) | ✓ (Teams only) | ✓ | — | — |
| Mobile “All sections” | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Footer attribution | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

Notes carried forward from §1: the only role gate the Dashboard component itself enforces is `canAccessDashboardSurface`, which currently always returns true. Any “role” differentiation above is therefore driven by capabilities (`workflow.pipeline`) and the participant-work surface gate, **not** by an explicit team role check inside the dashboard.

---

## 6. Card-content reference (one-line summary per visible block)

| Block | Title (i18n key) | Subtitle / first line | Icon | Background |
|---|---|---|---|---|
| Weather (with location) | n/a (header is the weather row itself) | Greeting + name + nextDue | `CloudRain` (passed as `sectionIcon`, currently unused inside WeatherCard – the card uses inline `WeatherIcon`) | gradient from `getWeatherCardGradient(weatherData)` |
| Greeting fallback | greeting + name | yellow next-due button or `nothingDueSoon` | — | default DrawerSection |
| Upcoming | `dashboard.upcoming.title` / `dashboard.upcoming.description` | participant + tree rows or `dashboard.nothingScheduled` | `CalendarClock` | default + Bell action chip |
| Tutorial | `dashboard.tutorialCard.title` / `…description` | next tutorial title + short desc, progress bar text | `GraduationCap` | inner `bg-card` button |
| AI Wizard | inline (no DrawerSection) | tip body + `dashboard.askMore` | `Sparkles` (decorative) | `--stormy-teal` solid |
| Your Trees | `dashboard.yourTrees.title` / `…description` | counts row + health grid + trend grid + CTA | `TreePine` | default DrawerSection |
| Work this week (Teams only) | `dashboard.workThisWeek` | up to 5 job rows (title + date), then `viewAllJobs` link | `Shovel` | default DrawerSection |
| Plan & account | `dashboard.planAndAccount` + tier badge | features-compact line + organization line | `CreditCard` | clickable DrawerSection |
| Quick actions | `dashboard.quickActions` | 6–8 icon tiles | `Zap` | default DrawerSection |
| Mobile Upgrade | `dashboard.upgradeSection` (fb `Upgrade`) | 1–3 upgrade tiles with glow icons | `Sparkles` | default DrawerSection (mobile only) |
| Mobile All sections | `dashboard.allSections` | grouped icon-tile grids per category | `Zap` | default DrawerSection (mobile only) |
| Footer | — | `dashboard.weatherDataBy` link to visualcrossing.com | `ExternalLink` | none |

---

## 7. Notable implementation details & risks

1. **`canAccessDashboardSurface` is a no-op.** The defense-in-depth gate at lines 119–123 / 389–391 of `dashboard.tsx` currently has no effect. The neighboring comment claims the dashboard is hidden for Teams non-owner/admin members; it is not. Either re-implement the function or update the comment to avoid future false-positive incidents.
2. **Plus tier weather fall-through.** `getWeatherCardTier` has no explicit branch for `"Plus"`, so Plus users get the **`home`** weather variant (no advisories, no feels-like/gusts/humidity, no `weatherRecommendations` ⇒ no AI tip pri 2/7). If Plus is ever meant to upgrade the WeatherCard, the change is one line in `lib/utils/tier-utils.ts`.
3. **Two “today” boundaries.** Participant rows use UTC-day chip semantics (to agree with the Worker Agenda drawer); tree rows use local-day. Documented in source — surface as a known semantic gap when reasoning about cross-drawer parity.
4. **Inbox badge math is partial.** `inboxUnreadCount` = `notificationsUnread + messagesUnread`. The mobile dot also OR-s in `pendingAccessRequestsCount`, but the bell on the Upcoming header does not. Owners with pending access requests but zero notifications/messages will see the dot on mobile and *no* badge on the bell.
5. **AI tip i18n drift.** Pri 2 and 7 emit raw `recommendation.title` / `recommendation.message` strings (from `computeRecommendations`). Those bypass the `t(...)` flow, so localization of those tips depends on `computeRecommendations` itself producing locale-correct text.
6. **Add-tree limit is a soft block.** The limit check fires on the dashboard “Add tree” buttons (`Add tree` quick action and the empty-state CTA in Your Trees), but does **not** appear on participant or work-this-week rows — those navigate without limit checks because they don’t create trees. This is fine, just worth noting that limit enforcement lives in `handleAddTreeClick` only.
7. **Work this week and Plan & account** read `job.schedule.startDate` defensively for both Firestore `Timestamp` (with `toDate()`) and POJO `{ seconds }` shapes. Any new job source that uses neither shape would silently break the date filter (`startDate.toDate is not a function`). Same pattern is used for `lastAssessment` in the “not assessed in 12 months” calculation.
8. **`organizationId` fallback chain** ends at `user.uid`. For a brand-new account with no `organizationId` and no `organization.id`, queries scope to the user’s uid — which matches Firestore’s personal-org convention but means the dashboard *will* render data for a user whose org doc has not yet been provisioned.

---

## 8. File map

```
app/drawers/dashboard.tsx                           # main component (706 lines)
app/drawers/dashboard/index.ts                       # barrel re-export
app/drawers/dashboard/DashboardUpcomingList.tsx      # block 2 body
app/drawers/dashboard/DashboardTutorialCard.tsx      # block 3
app/drawers/dashboard/DashboardAiWizardCard.tsx      # block 4
app/drawers/dashboard/DashboardYourTreesSection.tsx  # block 5
app/drawers/dashboard/DashboardTrendSummary.tsx      # nested in block 5
app/drawers/dashboard/DashboardMobileDashboardSections.tsx  # block 9

lib/dashboard-surface-access.ts                      # currently always-true gate
lib/utils/dashboard-upcoming.ts                      # tree-derived upcoming items + work-item merge + today filter
lib/utils/dashboard-trends.ts                        # trendSummary
lib/utils/dashboard-ai-tip.ts                        # priority-ordered AI tip
lib/utils/tier-utils.ts                              # effectiveTier / weatherCardTier
lib/participant-work-surface-access.ts               # canAccessParticipantWorkSurface
hooks/useTeamsOnlyAccess.ts                          # capability workflow.pipeline
lib/drawers.ts                                       # registration + visibleForTiers
lib/drawer-registry.ts                               # mobile group/upgrade option helpers
components/weather/WeatherCard.tsx                   # block 1 body (tier=home|pro|team)
```
