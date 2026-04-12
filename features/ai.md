# AI Features

Groundzy includes AI-powered species identification and conversational AI (AI Wizard Chat).

## Overview

| Feature | Drawer ID | Description |
|---------|-----------|-------------|
| Identify | `ai-identifying-wand` | Species identification from photos |
| AI Chat | `ai-chat` | Conversational AI (Gemini) |

**Tier**: Home, Plus, Pro, Teams. AI usage limits apply (see profile/limits). Access via More drawer, dashboard, or tree-add flow.

## Species Identification (`ai-identifying-wand`)

- **Location**: `app/drawers/ai-identifying-wand.tsx`, `components/wizard/SpeciesIdentifierWizard.tsx`
- **Flow**:
  1. User uploads photo(s)
  2. PlantNet API (client-side) or server-side identification
  3. Results shown; user selects species
  4. Can return to `tree-add` with species + photos pre-filled, or to `edit-tree` if opened from there
- **Store**: `identifying-wand-store` – `openedFromTreeAdd`, `returnToEditTreeId`, `applySpeciesAndReturnToTreeAdd`, `consumePendingIdentifiedSpecies`, `consumePendingIdentifiedPhotos`
- **API**: `POST /api/species/identify/reserve` (reservation/slot)
- **Client**: `lib/plantnet-client.ts` – `NEXT_PUBLIC_PLANTNET_API_KEY`
- **Tree limit check**: If at limit, toast + upgrade CTA before navigating to tree-add

## AI Chat (`ai-chat`)

- **Location**: `app/drawers/ai-chat.tsx`
- **Backend**: `POST /api/ai/chat` – Google Gemini (`GOOGLE_GEMINI_API_KEY` or `GEMINI_API_KEY`)
- **Persistence**: Firestore `ai_chats/{chatId}`, `ai_chats/{id}/messages/{messageId}`
- **Context**: `buildFullContextForApi()` – user's trees, weather, location, etc.
- **Suggestions**: `buildDynamicSuggestions()` – contextual follow-up prompts
- **Attachments**: Images (base64 or URLs), max per message from `MAX_ATTACHMENTS_PER_MESSAGE`
- **Gallery**: Can save images to user gallery (`users/{userId}/gallery/`)
- **Chat list**: Sidebar with chat history, create new, delete
- **Usage limit**: `useAiUsage()` – when limit reached, `isUsageLimitError` on assistant message, upgrade CTA

## AI Tree Summary

- **API**: `POST /api/ai/tree-summary`
- Generates summary for a tree (e.g. for sharing, reports)
- Uses Gemini

## AI Usage

- **API**: `GET /api/ai/usage`
- **Hook**: `useAiUsage()`
- Tracks message count, limits by tier

## Context Builder (`lib/ai-chat/context-builder.ts`)

Builds context for API from:

- User's trees (count, health summary)
- Weather (current conditions)
- Location (map center, address)
- User preferences

## Attachments (`lib/ai-chat/attachments.ts`)

- `validateAttachment()` – type, size
- `fileToBase64()`, `fileFromUrl()` – for sending
- Storage: Firebase Storage bucket from `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET`

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `GOOGLE_GEMINI_API_KEY` / `GEMINI_API_KEY` | AI Chat, Tree Summary |
| `NEXT_PUBLIC_PLANTNET_API_KEY` | Species ID (client-side; add domain to PlantNet) |
