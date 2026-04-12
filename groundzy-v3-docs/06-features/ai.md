# AI (identify & chat)

## What it does

- **Species identification:** Photo-based ID flow (`ai-identifying-wand`, `SpeciesIdentifierWizard`), optional PlantNet client, reservation API.
- **AI Chat:** Gemini-powered conversational UI (`ai-chat`), persisted threads in Firestore, context from trees/weather/location.
- **Tree summary:** `POST /api/ai/tree-summary` for generated summaries.
- **Usage limits:** Tier-based; `GET /api/ai/usage`, `useAiUsage()`.

## Who uses it

**Home–Teams** per `docs/features/ai.md`; limits stricter on lower tiers.

## Data involved

- **Firestore:** `ai_chats/{id}` + `messages` subcollection (`types/ai-chat.ts`).
- **API:** `/api/ai/chat`, `/api/ai/tree-summary`, `/api/ai/usage`, `/api/species/identify/reserve`.
- **User prefs:** `saveAiChatHistory` in `UserPreferences` (`lib/firebase/firestore.ts`).
- **Storage / gallery:** Attachments may reference user gallery paths.

## UI patterns

Wizard full-bleed drawer exception; chat uses **ScrollArea** for thread (`docs/audits/drawer-shell-classification.md`). **Identifying wand store** for handoff to tree-add/edit.

## Dependencies

- Google Gemini API keys (env)
- Optional PlantNet key
- Trees (context), weather context builder in chat API
- Tier limits messaging

## Inconsistencies & overlaps

- **AI** context pulls **weather** + **trees**—overlaps **weather.md** and **trees.md** without a single “context” entity in Firestore.
- **Species ID** overlaps **trees** species catalog and **explore** species UI—multiple entry points to same taxonomy.
