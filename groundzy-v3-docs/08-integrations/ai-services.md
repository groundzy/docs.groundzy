# AI & related APIs

## What it does

**Generative AI** (Google Gemini) for chat and tree summaries; **species identification** (PlantNet + reservation route); **usage** tracking. **Weather** and **Places** use separate providers but follow the same “server or client + API key” pattern.

---

## Google Generative AI (Gemini)

| Use | Where |
|-----|--------|
| AI Chat | `POST /api/ai/chat` — `@google/generative-ai` |
| Tree summary | `POST /api/ai/tree-summary` |
| Usage stats | `GET /api/ai/usage` |

**Env:** `GOOGLE_GEMINI_API_KEY` or `GEMINI_API_KEY` (`docs/reference/environment-variables.md`).

**Persistence:** Firestore `ai_chats` + messages (via API routes using Admin SDK).

**Risks:** API cost scales with tokens; keys must stay server-side for routes; rate limits / safety policies are provider-side.

---

## PlantNet (species identification)

| Use | Where |
|-----|--------|
| Client-side identification | `lib/plantnet-client.ts` — `NEXT_PUBLIC_PLANTNET_API_KEY` |
| Reservation / slots | `POST /api/species/identify/reserve` |

**Risks:** **Public key** in browser — domain allowlist on PlantNet; abuse caps; not a substitute for licensed arborist judgment (product copy).

---

## Weather APIs (supporting data, not “AI” in name)

| Provider | Env | Route / use |
|----------|-----|-------------|
| **Visual Crossing** | `VISUALCROSSING_API_KEY` | `GET /api/weather/timeline` |
| **WeatherAPI.com** | `WEATHERAPI_KEY` | `GET /api/weather/current` |

**Risks:** External availability; server-side caching (`lib/weather/cache.ts` pattern) reduces calls; keys server-only.

---

## Google Places (nearby businesses)

| Use | Where |
|-----|--------|
| Hire a Pro / tree services | `GET /api/places/nearby-tree-services` |

**Env:** `GOOGLE_PLACES_API_KEY` or aliases (`GOOGLE_MAPS_API_KEY`, `GOOGLE_API_KEY`).

**Risks:** Quota/billing on Google Cloud; key restriction by API in Google console.

---

## Cross-cutting risks

| Fragmentation | Multiple vendors (Google AI vs PlantNet vs Visual Crossing vs Places) — separate keys, dashboards, and failure modes. |
| **Missing abstraction** | No single “AI gateway” in repo — each route owns its client and errors. |

## Related

- `Groundzy v3/06-features/ai.md`, `weather.md`, `hire-a-pro.md`
- `types/ai-chat.ts`
