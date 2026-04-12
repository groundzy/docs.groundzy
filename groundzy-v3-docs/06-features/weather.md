# Weather

## What it does

Location-based **current** and **timeline** weather, **tree impact** scoring, and **recommendations** (watering, freeze, storm inspection, **job reschedule**, etc.). Shown in **weather** drawer and embedded in dashboard/AI context as applicable.

## Who uses it

All tiers; **UX depth** varies (e.g. Pro 12-hour forecast mentioned in `docs/features/weather.md`, `getWeatherCardTier` in `lib/utils/tier-utils.ts`).

## Data involved

- **No** primary Firestore **weather** entity—**fetched** via `/api/weather/timeline` and `/api/weather/current` (Visual Crossing, WeatherAPI.com).
- **H3** cell caching on server (`lib/weather/cache.ts` pattern).
- Types: `lib/weather/types.ts`, recommendations in `lib/weather/recommendations.ts`.

## UI patterns

**drawer:** `weather` → `weather-impact.tsx`; gradient styling per visual inventory. **Dashboard** weather card tier variants.

## Dependencies

- Map center or GPS for location
- API keys env
- **Workflow:** `job_reschedule` recommendation type links to **workflow.md**

## Inconsistencies & overlaps

- **Weather “timeline”** is **external forecast**—not the same as **tree Activity timeline** or **event history** (`Groundzy v3/05-data/event-system.md`)—naming collision for users/docs.
- **AI chat** ingests weather context—**overlap** with **ai.md** without shared persistence layer.
