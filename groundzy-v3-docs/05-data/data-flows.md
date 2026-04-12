# Data Flows

How data **moves** through the app for major operations. Client paths use **Firebase SDK** + **React Query** unless noted.

---

## 1. Create a tree

1. User places marker / fills **Add Tree** form (`components/trees/add-tree-form.tsx` — large orchestration).
2. **Write** `trees/{treeId}` with `organizationId` / `databaseCode`, species, location, optional `propertyId`, `clientId`, `zoneId`.
3. Optional **tree number** counter update (`tree_number_counters`).
4. **History** entries added via `addTreeHistoryEntry` → updates embedded `history` and/or **`tree_events`** subcollection (`lib/firebase/firestore.ts`).
5. **Dual-write** to `work_items` when prefs allow and `organizationId` present (`fireAndForgetSyncTreeHistory` in `lib/firebase/work-items.ts`).
6. **Invalidate** React Query keys for trees / tree detail.

**Break points:** Large form = many side effects in one flow; dual-write can fail silently (logged).

---

## 2. Request → Quote → Job → Invoice

1. **Create Request** → `requests` doc; optional `workItemIds` / mirror sync.
2. **Convert / create Quote** → `quotes` doc with `requestId`, `propertyId`, line items; navigation often prefilled via URL params (`docs/features/current-workflow-audit.md` historically described nav-heavy flows; upgrade docs added conversion functions).
3. **Job** from quote or request → `jobs` doc with `quoteId?`, `requestId?`, `treeIds`, pricing.
4. **Invoice** requires **`jobId`** → `invoices` doc with line items, `quoteId?`.
5. Each stage may **upsert** `work_items` mirror rows (`fireAndForgetSync*`).

**Break points:** If conversion backlinks or status sync are incomplete, UI and reports can disagree; **work_items** can duplicate CRM state.

---

## 3. Update a property

1. Load property via Query hook.
2. **updateProperty** → `properties/{id}`; may touch `clientId`, `zoneId`, address.
3. **Trees** on property not auto-moved unless app logic does so—relationship is **foreign key**, not cascading FK.

---

## 4. AI chat

1. User sends message in AI Chat drawer.
2. **Persist** thread in `ai_chats/{chatId}` with subcollection `messages` (`types/ai-chat.ts`).
3. **API call** `POST /api/ai/chat` (or similar) with **Gemini** — server uses API keys.
4. Usage may be tracked via `/api/ai/usage` and limits per tier.

**Flow:** Firestore (chat storage) + HTTP (model inference); not a single unified “AI entity” beyond `AiChat` + messages.

---

## 5. Weather

1. Client **GET** `/api/weather/timeline` or `/api/weather/current` with lat/lng.
2. Server calls **external** weather provider; may use **H3** cache (`lib/weather/cache.ts` pattern).
3. **No** persistent Firestore weather record in the surveyed rules.

---

## 6. Billing / subscription

1. **Stripe** Checkout / Portal via **API routes** (`app/api/stripe/*`).
2. **Webhook** `/api/stripe/webhook` updates **user** or **team** subscription fields.
3. **Client** reads user doc → `getEffectiveSubscriptionTier` gates UI (`lib/utils/tier-utils.ts`).

---

## 7. Where flows don’t connect cleanly

| Issue | Description |
|-------|-------------|
| **Activity tab** | Merges `tree.history` + `work_items` in UI only when **preference** flag set—data layer still multiple sources. |
| **Hire a Pro** | `pro_contact_requests` + external Places — separate from CRM Request. |
| **Community vs private tree** | `tree_public`, `tree_public_summaries`, `community_posts` — parallel public graph. |

---

## Related

- [`relationships.md`](./relationships.md)
- [`event-system.md`](./event-system.md)
