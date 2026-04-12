# Intelligence / Alerts / Notifications — Implementation Plan (Groundzy)

**Audience:** Engineering  
**Scope:** Event-first **derived** intelligence (`intelligence.*`), evaluation triggers, dedupe, provenance, in-app notification delivery, tier gating — aligned with existing `app.groundzy` and [`docs/architecture/intelligence/FILE-STRUCTURE.md`](./intelligence/FILE-STRUCTURE.md).  
**Out of scope for this document:** Product marketing copy, non-Groundzy infrastructure.

This plan **does not** assume features that are absent from the repo today (no SMS provider integration, no Firebase Cloud Functions in this workspace, no email digest pipeline beyond transactional Resend patterns). Where those appear in product docs, they are listed as **future phases** with explicit dependencies.

---

## 1. Current state (verified in repo)

### 1.1 Groundzy events (primary pipeline)

| Item | Location / fact |
|------|------------------|
| **Collection** | `groundzy_events` — rules: client **denied** (`firebase/firestore.rules`); writes via **Admin SDK** only |
| **Schema** | `lib/groundzy/events/schema/system.ts` — `GroundzyEventType` is **only** `workflow.*` and `tree.*` |
| **Append path** | `appendGroundzyEvent` in `lib/groundzy/server/append-event.ts` — **Firestore transaction**, `groundzy_event_idempotency` keyed by `idempotencyDocId(organizationId, idempotencyKey)` |
| **HTTP entry** | `POST /api/groundzy/events/append` — **requires Firebase `idToken`**; resolves actor from token; runs `assertCanAppendEvent` (`lib/groundzy/policy/can-append-event.ts`) |
| **Projections** | `lib/groundzy/projections/handlers/index.ts` — registry **only** for workflow + tree timeline types |

**Implication:** Derived events **cannot** be appended through the existing public append API without a **human** actor unless you extend the server (see §4.2).

### 1.2 Weather (pure evaluation, mostly pull)

| Item | Location / fact |
|------|------------------|
| **Tree impact** | `lib/weather/tree-impact.ts` — `computeTreeImpact(normalized, unitGroup)` |
| **Recommendations** | `lib/weather/recommendations.ts` — `computeRecommendations(weather, trees, jobs?, options)`; uses `TreeForRecommendations` (health, species care, measurements) |
| **API** | `GET /api/weather/timeline` — normalizes Visual Crossing, computes `treeImpact` server-side (`app/api/weather/timeline/route.ts`) |
| **Current** | `GET /api/weather/current` — separate provider |
| **Client usage** | `dashboard.tsx`, `weather-impact.tsx` call `computeRecommendations` **client-side** with loaded trees + weather |

**Implication:** Logic is **reusable**; there is **no** persisted “weather snapshot” id in Firestore today and **no** scheduled refetch job in-repo.

### 1.3 Notifications (in-app only)

| Item | Location / fact |
|------|------------------|
| **Collection** | `notifications/{id}` |
| **Client** | `lib/firebase/notifications.ts` — `AppNotification`: `userId`, `type`, `title`, `message`, `actionUrl?`, `read`, timestamps |
| **Rules** | Read own; **create false**; update **only** `read → true` |
| **Writers found** | `app/api/dev/simulate-payment/route.ts`, `app/api/dev/create-user/route.ts` (dev); **no** production Stripe → notification path in the same grep set — treat billing notifications as **documented elsewhere / verify before relying** |

**Implication:** Production pattern is **Admin SDK** writes; new intelligence writers follow the same. No `sourceEventId` / `severity` fields in code today.

### 1.4 Tiers & capabilities

| Item | Location / fact |
|------|------------------|
| **Tier helpers** | `lib/utils/tier-utils.ts` — `getEffectiveSubscriptionTier`, paid checks |
| **Capabilities** | `lib/capabilities.ts` — `CapabilityKey` set for **CRM**, **workflow**, **map jobs**; **no** `intelligence.*` or `notifications.*` keys yet |
| **Entitlements** | `lib/groundzy/entitlements/resolve.ts` — derived from user doc; used in append path |

**Implication:** Tier gating for intelligence should **extend** `CapabilityKey` + `getCapabilitiesForTier` (or a dedicated small module) to avoid duplicating tier logic in UI and server.

### 1.5 Background / scheduled compute

| Finding | |
|---------|--|
| **No** `firebase/functions` tree in this repo; **no** `cron` config found in-app | Scheduled evaluation is **not** implemented. Options: Vercel/ Firebase App Hosting cron → `route.ts`, Cloud Scheduler → HTTP, or future Cloud Functions repo. |

---

## 2. Reusable pieces (use as-is or thin wrappers)

1. **`computeTreeImpact` / `computeRecommendations`** — keep as **pure** functions; call from server evaluator with normalized DTOs (same shapes as `lib/weather/types.ts`).
2. **`GET /api/weather/timeline`** — reuse normalization + cache (`lib/weather/cache.ts`, H3) for any server-side “fetch forecast for lat/lng”.
3. **Idempotency pattern** — mirror `groundzy_event_idempotency` + transaction pattern from `append-event.ts` for intelligence writes (same key semantics: stable idempotency key per logical emission).
4. **Tree fields** — `types/tree.ts` exposes `health.overallStatus`, `health.riskLevel`, `lat`/`lng` for matching docs and vertical-slice conditions.
5. **Docs** — payload shapes, tier matrix, vertical slice: [`docs/groundzy-v3-docs/05-data/derived-events-payload-schema.md`](../groundzy-v3-docs/05-data/derived-events-payload-schema.md), [`../groundzy-v3-docs/09-business/intelligence-tier-matrix.md`](../groundzy-v3-docs/09-business/intelligence-tier-matrix.md), [`../groundzy-v3-docs/99-reference/vertical-slice-intelligence-v1.md`](../groundzy-v3-docs/99-reference/vertical-slice-intelligence-v1.md).

---

## 3. Missing architecture (to build)

| Concern | What to add |
|---------|-------------|
| **Derived event types** | Extend `GroundzyEventType` + Zod payloads + (optional) projection handlers for read models |
| **System actor writes** | `appendGroundzyEvent` **requires user idToken** — intelligence needs **`appendIntelligenceEvent` via Admin SDK** (or service account) with documented **system `actorUserId`** and **no** `assertCanAppendEvent` for human actions (replace with “server-only” checks) |
| **Evaluation triggers** | At minimum: **HTTP-triggered** evaluation (cron or manual); optional: Firestore trigger on `trees` updates (cost/risk tradeoff) |
| **Dedupe** | Persist `dedupeKey` on event payload + **same idempotency collection** pattern or a dedicated `intelligence_dedupe` collection — must align with duplicate evaluation runs |
| **Provenance** | `causationEventId` / `correlationId` on derived events; link `notifications.sourceEventId` → `groundzy_events` id |
| **Delivery** | In-app: extend `notifications` docs + optional `lib/firebase/notifications.ts` fields; **Email/SMS**: not in repo — phase later with Resend + provider |

---

## 4. Design decisions & insertion points

### 4.1 Event schema

**Files:** `lib/groundzy/events/schema/system.ts`, `lib/groundzy/events/schema/*.ts` (new `intelligence.ts`), `lib/groundzy/events/validate.ts`.

- Add types such as `intelligence.alert_triggered` (and optionally `intelligence.rule_evaluated`, `intelligence.alert_resolved`) per [`derived-events-payload-schema.md`](../groundzy-v3-docs/05-data/derived-events-payload-schema.md).
- Extend `GroundzySubject` — tree-level alerts use `{ entityType: "tree", id: treeId }` (already valid).

### 4.2 Append path (critical fork)

**Do not** route system derived events through `POST /api/groundzy/events/append` with a fake user token.

**Preferred approach:**

- New module e.g. `lib/groundzy/server/append-intelligence-event.ts` that:
  - Uses **Admin SDK** `adminDb` + **transaction**
  - Writes `groundzy_events/{eventId}` with same shape as existing docs (`schemaVersion`, `type`, `organizationId`, `actorUserId` = system uid constant, `correlationId`, `causationEventId?`, `subject`, `payload`, `createdAt`, `idempotencyKey`)
  - Writes idempotency row **or** reuses `groundzy_event_idempotency` with a **namespaced** idempotency key prefix to avoid collisions with user commands: e.g. `intel:${organizationId}:${dedupeKey}`
  - **Skips** workflow sequence allocation and **skips** tree/workflow projection handlers unless you add explicit handlers for intelligence types

**Optional:** Internal `POST /api/internal/intelligence/evaluate` secured by **CRON_SECRET** / OIDC — calls the above. No end-user token.

### 4.3 Projections

**Files:** `lib/groundzy/projections/handlers/index.ts`, new handler if materialized view needed.

- **Minimal slice:** no new projection collection — only event + notification doc.
- **Later:** optional `intelligence_alerts` / user inbox index if queries become heavy.

### 4.4 Notifications

**Files:** `lib/firebase/notifications.ts` (types + optional new fields in mapper), writers in **server** code only.

Add fields (when implementing):

- `sourceEventId?: string`
- `severity?: 'info' | 'recommendation' | 'warning' | 'critical'`
- `organizationId?: string` (for rules)

**Rules:** extend `firestore.rules` only if new client-writable fields are needed; **prefer** keeping client `update` to `read` only.

### 4.5 Tier gating

**Files:** `lib/capabilities.ts`, `lib/utils/tier-utils.ts` (or new `lib/intelligence/tier.ts`).

- Add capability keys, e.g. `intelligence.evaluation.basic`, `intelligence.channels.email` — **wire** to same tier source as UI (`getEffectiveSubscriptionTier`).
- Evaluator **reads tier** for `organizationId` (owner user or team) before running rules.

### 4.6 Evaluation triggers (minimal vertical slice)

**No cron required for v0** if product accepts **on-demand** evaluation:

- **Option A:** Extend existing user journey — e.g. after weather loads on **dashboard**, server already has trees + could call an internal API **once per session** (rate-limit) — **couples** to session; weaker than push.
- **Option B (recommended for “system-driven”):** **Scheduled** HTTP route (hosting cron) e.g. `GET /api/internal/intelligence/run?scope=daily` with secret header — loads orgs/users + tree list + fetches weather per region — **heavy**; scope to **one org** or **pilot users** first.
- **Option C:** Firestore **`onWrite`** for `trees` — only re-evaluates one tree; does **not** solve weather-only triggers without additional signals.

For **minimal slice** aligned with storm × weak tree: **cron + batch** or **manual trigger from dev/admin route** is honest; **true push without any scheduler** is not available in-repo today.

---

## 5. Data model changes (incremental)

| Store | Change |
|-------|--------|
| `groundzy_events` | New `type` values; payload includes `ruleId`, `version`, `evaluation`, `outputs`, `dedupeKey`, `inputPointers` |
| `groundzy_event_idempotency` | Same collection; **key namespace** for intelligence |
| `notifications` | Optional `sourceEventId`, `severity`, `organizationId`; index if querying by `sourceEventId` |
| `users` / `teams` | **Future:** preferences for digest/quiet hours — **not** required for slice |

---

## 6. Phased build order

### Phase 0 — Preconditions (1–2 PRs)

- [ ] Document **system user id** for `actorUserId` (env-specific Firebase Auth uid or dedicated “service” doc — **do not** invent without Infra).
- [ ] Add Zod schemas + **TypeScript-only** `GroundzyEventType` entries (no behavior yet).
- [ ] Add `appendIntelligenceEvent` Admin path + unit test with emulator or mock transaction.

**Risk:** Transaction size if batching many trees — **batch in chunks**.

### Phase 1 — Minimal vertical slice (prove value)

**Goal:** One **rule**: storm/high-wind signal + tree with elevated risk (`riskLevel` / `overallStatus` per `TreeForRecommendations` / `recommendations.ts` patterns) → **one** `intelligence.alert_triggered` + **one** `notifications` row per user.

**Steps:**

1. Implement **evaluator** function: input `(organizationId, treeId list or single tree)`, fetch weather via **shared** normalize from timeline API internals (refactor `timeline/route` if needed to import shared builder).
2. Map wind + tree state using thresholds **aligned** with `computeRecommendations` storm logic (`storm_inspection` / high wind) to avoid two conflicting definitions — extract shared **threshold constants** if duplicated.
3. Resolve **recipient user id** (tree owner: `databaseCode` / `organizationId` on `trees` — match existing app patterns for “who owns this tree”).
4. Call `appendIntelligenceEvent` then write `notifications` with Admin SDK (same transaction **or** notifications after event with compensating logging — **prefer single transaction** if Firestore limits allow).
5. **Dedupe:** `dedupeKey = f(ruleId, treeId, dateBucket)` — store in payload + idempotency key.

**Explicitly not in Phase 1:** Email, SMS, map overlay, workflow job creation, Firestore triggers on all trees.

**Trigger for Phase 1:** **Manual** `POST /api/internal/...` or **cron** hitting secured route — **document** which exists after implementation.

### Phase 2 — Provenance & inbox UX

- [ ] Surface `sourceEventId` in `ActivityTab` / debug (internal) or “why” link when product ready.
- [ ] Extend `AppNotification` type in client; optional filter by `severity`.

### Phase 3 — Tier gating

- [ ] Gate evaluator **rule sets** by `getCapabilitiesForTier` / new keys.
- [ ] Align with [`intelligence-tier-matrix.md`](../groundzy-v3-docs/09-business/intelligence-tier-matrix.md) (product sign-off).

### Phase 4 — Email digest (optional, only if Resend + templates exist)

- [ ] Batch notifications → Resend (`lib/email.ts` pattern) — **requires** unsubscribe storage and legal review.

### Phase 5 — SMS / advanced routing

- **No implementation** until provider + consent + tier — **out of repo scope today**.

---

## 7. Risks & mitigations

| Risk | Mitigation |
|------|------------|
| **Dual append paths** (user vs system) drift | Shared `buildEventDoc` helper; single schema version |
| **Cost** — weather API calls × trees | H3 batching, cache TTL (`lib/weather/cache.ts`), rate limits per cron |
| **Spam** — repeated evaluations | Dedupe + idempotency; optional `expiresAt` on alerts |
| **Authorization** — wrong user sees alert | Resolve recipient using same rules as tree read access; test with shared trees |
| **append-event.ts size** | Keep intelligence out of main user transaction unless merged carefully |

---

## 8. Verification checklist (before “done”)

- [ ] Event appears in `groundzy_events` with correct `type` and `subject`
- [ ] Duplicate run returns **duplicate: true** or no second notification (idempotent)
- [ ] Firestore rules: client still cannot forge intelligence events
- [ ] Tier: Home vs Pro behavior matches matrix (if gated)
- [ ] No regression in `POST /api/groundzy/events/append` for existing types

---

## 9. Reference doc index

| Doc | Use |
|-----|-----|
| [`architecture/intelligence/FILE-STRUCTURE.md`](./intelligence/FILE-STRUCTURE.md) | Full markdown map |
| [`groundzy-v3-docs/05-data/derived-events-payload-schema.md`](../groundzy-v3-docs/05-data/derived-events-payload-schema.md) | Payloads |
| [`groundzy-v3-docs/05-data/evaluation-runtime.md`](../groundzy-v3-docs/05-data/evaluation-runtime.md) | Triggers & fan-out |
| [`groundzy-v3-docs/07-systems/notification-projection-model.md`](../groundzy-v3-docs/07-systems/notification-projection-model.md) | Notifications ↔ events |
| [`reference/intelligence-event-types.md`](../reference/intelligence-event-types.md) | Enum sync with code |

---

**Document status:** Living — update when first internal route ships and when `GroundzyEventType` in code is extended.
