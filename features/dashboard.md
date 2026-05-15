# Dashboard Feature

The Dashboard is the main overview drawer, shown as the first item in the sidebar and bottom nav for Home, Plus, Pro, and Teams tiers.

## Overview

- **Drawer ID**: `dashboard`
- **Location**: `app/drawers/dashboard.tsx`
- **Tier**: Home, Plus, Pro, Small Team, Mid Team, Large Team, Enterprise

## Sections

### Greeting

Time-based greeting (morning, afternoon, evening, night) using `getGreetingKey()`.

### Quick Actions

- **Add Tree** – Opens `tree-add` drawer
- **Identify** – Opens `ai-identifying-wand` drawer
- **AI Wizard** – Opens `ai-chat` drawer
- **Profile** – Opens `profile` drawer
- **Team Settings** – Opens `team-settings` (Teams only)
- **Help** – Opens `help` drawer
- **Inbox** – Opens `contact-us` drawer
- **Upgrade** – Stripe checkout (Home/Plus users)

### Health Summary

- Health breakdown: excellent, good, fair, poor, critical, dead
- **Needs attention count**: poor + critical + dead
- **Healthy count**: excellent + good
- Links to filtered Map Items list (health filter)

### Weather Card

- **Location**: Map center or user location (reverse geocoded via Mapbox)
- **Data**: `useWeatherContext()` – fetches `/api/weather/timeline` (Visual Crossing)
- **Tier-based UX**:
  - **Home**: Basic conditions, current temp
  - **Pro**: 12-hour hourly forecast, tree impact
  - **Team**: Operational view, recommendations
- **Recommendations**: `computeRecommendations()` from `lib/weather/recommendations.ts`
- Opens full `weather` drawer on click

### Upcoming

- **Source**: `getDashboardUpcoming(trees, locale, speciesCatalog)` from `lib/utils/dashboard-upcoming.ts`
- Scheduled services/inspections from tree history
- Localized species names from catalog
- **Participant “today” rows**: When the signed-in user sees **My work** / participant items merged into the Upcoming card, rows for **request**, **quote**, and **job** are omitted if that stage is **off** in the team org’s `crmWorkflowProfile.stages` (`isWorkflowStageOnInProfile` in `app/drawers/dashboard.tsx`). Tree-scheduled upcoming rows are unchanged.

### Trend Summary

- **Source**: `getDashboardTrendSummary(trees)` from `lib/utils/dashboard-trends.ts`
- Health/risk trends over time

### AI Wizard Card

- **Source**: `getPrioritizedAiTip()` from `lib/utils/dashboard-ai-tip.ts`
- Contextual AI tips based on user state

### Tutorial Card

- Links to `tutorial` drawer

### Notifications & Messages

- **Unread notifications**: `useUnreadNotifications()`
- **Unread messages**: `useUnreadMessages()` (Inbox)
- Badge counts on Inbox/Profile

### Limits & Upgrade

- **Tree usage**: `useTreeUsage()` – used/limit
- **AI usage**: `useAiUsage()`
- **Plus checkout**: `usePlusCheckout()` for Home users
- **Pro checkout**: `useProCheckout()` for Plus users
- Copy from `lib/profile-copy.ts` (`LIMITS_COPY`)

## Data Dependencies

| Hook/Util | Purpose |
|----------|---------|
| `useAuth` | user, userDoc, organization |
| `useTrees` | trees list |
| `useTreeUsage` | tree limits |
| `useAiUsage` | AI limits |
| `useJobs` | jobs (upcoming) |
| `useWeatherContext` | weather timeline, recommendations |
| `useMapStore` | map center (location) |
| `useMapboxPlaceLabel` | resolved address for weather |
| `useUserPreferences` | units, timezone |
| `useUnreadNotifications` | notification count |
| `useUnreadMessages` | inbox count |
| `useSpeciesCatalog` | species names for upcoming |
| `useTeamsOnlyAccess` | Teams feature access |
