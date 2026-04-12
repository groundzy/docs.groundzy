# Weather Feature

Weather provides location-based conditions, forecasts, and tree-care recommendations.

## Overview

- **Drawer ID**: `weather`
- **Location**: `app/drawers/weather-impact.tsx`
- **Tier**: Home, Plus, Pro, Teams (UX varies by tier)
- **APIs**: Visual Crossing (timeline), WeatherAPI.com (current)

## Data Flow

1. **Location**: Map center or user GPS (`getCurrentPositionSafe`)
2. **Reverse geocode**: Mapbox `reverseGeocode()` for address label
3. **Timeline**: `GET /api/weather/timeline` – Visual Crossing (`VISUALCROSSING_API_KEY`)
4. **Current**: `GET /api/weather/current` – WeatherAPI.com (`WEATHERAPI_KEY`)
5. **Context**: `useWeatherContext()` – fetches timeline + recommendations

## Weather Types (`lib/weather/types.ts`)

| Type | Description |
|------|-------------|
| `WeatherLocation` | resolvedAddress, timezone, lat/lng |
| `WeatherCurrent` | temp, feelslike, conditions, humidity, windspeed, etc. |
| `WeatherHour` | Hourly data (Pro 12-hour forecast) |
| `WeatherDay` | Daily min/max, conditions, precip, etc. |
| `WeatherAlert` | event, headline, description, severity, start/end |
| `WeatherTimelineDTO` | location, current, days, hours?, alerts, treeImpact |
| `TreeImpact` | level (none/mild/moderate/high), summary |
| `WeatherRecommendation` | type, priority, title, message, relatedTreeId, etc. |

## Recommendation Types

From `lib/weather/recommendations.ts`:

| Type | Description |
|------|-------------|
| `watering` | Water trees (dry spell, high water needs) |
| `skip_watering` | Skip watering (rain expected) |
| `freeze_protection` | Protect young/vulnerable trees from frost |
| `irrigation_reminder` | Remind to irrigate (dry spell, no rain soon) |
| `storm_inspection` | Inspect trees after storm (high wind) |
| `job_reschedule` | Reschedule outdoor jobs (bad weather) |

## Recommendation Logic

- **Young trees**: Estimated age < 3 years
- **Elevated risk**: poor/critical/dead health, high/extreme/imminent risk
- **Water needs**: From species `care.waterNeeds` (low/moderate/high)
- **Frost**: Temp ≤ 32°F (imperial) or 0°C (metric)
- **High wind**: Gust ≥ 30 mph or 13.4 m/s
- **Dry spell**: Next 7 days precip < 0.4 in or 10 mm
- **Rain chance**: Skip irrigation if any of next 3 days has precip prob ≥ 50%

## Tier-Based UX

| Tier | Dashboard WeatherCard | Weather Drawer |
|------|----------------------|----------------|
| Home | Basic conditions, current temp | Current + 7-day summary |
| Pro | 12-hour hourly, tree impact | Full timeline, hourly, tree impact, recommendations |
| Team | Operational view | Same as Pro + job reschedule |

## Components

- **WeatherCard**: `components/weather/WeatherCard.tsx` – compact card for dashboard
- **ConditionsCard**: `components/weather/conditions-card.tsx` – conditions display
- **Gradients**: `lib/weather/weather-card-gradients.ts` – `getWeatherCardGradient()`

## User Preferences

- **Units**: Imperial (F, in) vs Metric (C, mm) – `useUserPreferences().isImperial`
- **Timezone**: For sunrise/sunset, date labels – `useUserPreferences().timezone`

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `VISUALCROSSING_API_KEY` | Weather timeline |
| `WEATHERAPI_KEY` | Current weather |
| `NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN` | Reverse geocode for address |
