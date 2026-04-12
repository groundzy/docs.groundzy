# Dashboard

## What it does

Entry **home** drawer: quick actions, tree/health summaries, **weather** card, **upcoming** jobs/events, notifications/inbox entry, limits CTAs. May **merge** scheduled `work_items` into upcoming when user pref enabled (`mergeDashboardUpcomingWithWorkItems`).

## Who uses it

All tiers; content **depth** and widgets vary by tier (weather card tier mapping, workflow widgets Teams-only).

## Data involved

- Aggregations from **trees**, **user** prefs, **notifications**, **weather** API (via hooks), **work_items** when merge flag on.
- Not a single Firestore `dashboard` document—**computed** view.

## UI patterns

`dashboard` drawer with `DrawerShell`; tea-green accent patterns per visual inventory.

## Dependencies

- Nearly all domains superficially: trees, weather, billing limits, optional work_items
- Navigation store, inbox unread counts

## Inconsistencies & overlaps

- **Upcoming** can combine **embedded tree history** with **work_items**—same time axis, two sources (`dashboard.md` in docs/features + `lib/utils/dashboard-upcoming`).
- Overlaps **map** (spatial) vs **dashboard** (summary)—duplicate entry points to jobs/trees.
