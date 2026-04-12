# Drawer discipline & Phase C — handoff document

This document is a **standalone summary** of the drawer refactor initiative: **Discipline Phase 2** (rules + tooling + required extractions) and **Phase C** (heavy entry-file burn-down). Use it if you no longer have the original planning chat.

---

## 1. What problem we were solving

- **Early issue:** drawer entry files were huge blobs of UI.
- **Later / current issue:** entry files are **orchestration-heavy** — they centralize mode switches, hooks, navigation, and attachment/gallery wiring. The bottleneck is **centralized orchestration complexity**, not “messy JSX” alone.
- **Long-term direction:** entry components become **composition roots** (orchestration + wiring), not the primary home for large leaf UI blocks or duplicated layout.

**Hard constraint:** [`lib/drawers.ts`](../lib/drawers.ts) must keep **stable lazy import paths** to public entry components. New code lives in **colocated folders** under `app/drawers/<name>/` and is imported only from the existing entry file.

---

## 2. Phase 2 — discipline (rules + enforcement)

Phase 2 established **how** drawer work stays consistent over time.

| Item | Purpose |
|------|---------|
| **Binary Shell rule** | Every registered drawer is either **using `DrawerShell`** (from `@/components/drawer-layout`) or **listed as a documented exception** — no unnamed “third state.” See [`docs/drawer-shell-classification.md`](drawer-shell-classification.md). |
| **`npm run verify:drawers`** | Non-blocking **line-count warning** for top-level `app/drawers/*.tsx` files over **400 lines** (see [`scripts/drawer-pr-check-warn.mjs`](../scripts/drawer-pr-check-warn.mjs)). Exit code stays 0; treat output as a backlog signal. |
| **PR checklist** | [`docs/DRAWER_PR_CHECKLIST.md`](DRAWER_PR_CHECKLIST.md) — reviewers expect drawer PRs to respect Shell/scroll patterns where applicable. |
| **Retrospective** | [`docs/drawer-refactor-second-pass-retrospective.md`](drawer-refactor-second-pass-retrospective.md) — narrative of passes, inventories, gaps. |

**Required extractions (Phase 2 “no longer optional”)** — these were treated as **must-ship** so they would not linger as “someday”:

- **`AiChatComposer`** — `app/drawers/ai-chat/AiChatComposer.tsx`, used from `AiChatThread`.
- **`RequestFormAssigneesSection`** — `app/drawers/request-form/RequestFormAssigneesSection.tsx`.
- **`ContactUsGalleryPickerDialog`** — `app/drawers/contact-us/ContactUsGalleryPickerDialog.tsx` (shared thread + support gallery flows).

---

## 3. Phase C — line budget & burn-down

**Goal:** Drive down **orchestration entry** files over time toward a **soft ~400 line** target for `app/drawers/*.tsx` top-level entries (aligned with `verify:drawers`).

**Principles:**

- **One coherent vertical slice per PR** when possible.
- **Behavior-neutral** extractions unless a bugfix is explicitly scoped.
- **No new architecture layer** — presentational components + props/callbacks from parent.
- **Parent keeps hooks and orchestration** until a slice is clearly bounded.
- If a PR changes a drawer **root layout**, update [`docs/drawer-shell-classification.md`](drawer-shell-classification.md) in the same PR.

**Suggested priority order for the “big five” interaction drawers:**

`contact-us` → `request-form` → `view-zone` → `ai-chat` → `profile`

(Other top-level drawers such as `dashboard.tsx` or `client-form.tsx` may rank **higher by raw line count** in `verify:drawers`; prioritize by product initiative.)

---

## 4. What was implemented (artifact inventory)

### 4.1 Contact us — `app/drawers/contact-us.tsx`

Colocated components under `app/drawers/contact-us/`:

| File | Role |
|------|------|
| `ContactUsThread.tsx` | Thread header, messages, loading |
| `ContactUsThreadComposer.tsx` | Composer / reply / attachments |
| `ContactUsGalleryPickerDialog.tsx` | Shared gallery picker dialog |
| `ContactUsInboxHome.tsx` | Default inbox (Activity / Messages, shortcuts, list) |
| `ContactUsSupportInbox.tsx` | Support inbox list |
| `ContactUsMessageUserPanel.tsx` | Start conversation by username |
| `ContactUsShareTreePanel.tsx` | Share tree flow |
| `ContactUsAccessRequestDetailPanel.tsx` | Single access request |
| `ContactUsAccessRequestsListPanel.tsx` | Access requests list |
| `ContactUsContactSupportPanel.tsx` | **Contact Support** form + gallery wiring for support attachments |
| `ParticipantLabel.tsx`, `InboxGalleryGrid.tsx` | Supporting UI |

Supporting: `app/drawers/contact-us-formatters.ts` (shared formatters).

### 4.2 Request form — `app/drawers/request-form.tsx`

Colocated under `app/drawers/request-form/`:

| File | Role |
|------|------|
| `RequestFormItemServicesCollapsible.tsx` | Per tree/zone service toggles |
| `RequestFormAssigneesSection.tsx` | Assignees popover |
| `RequestFormTreesAndZonesSection.tsx` | Trees/zones selection + map actions |
| `RequestFormScheduleSection.tsx` | Schedule card |
| `RequestFormIntakeSection.tsx` | Intake / priority card |
| `RequestFormClientPropertySection.tsx` | Client, contact, property selects |
| `RequestFormRequestDetailsSection.tsx` | Title, service details, trees/zones **slot**, description |

Supporting: `app/drawers/request-form-constants.ts`, `app/drawers/RequestFormMapMarkerThumb.tsx`.

### 4.3 View zone — `app/drawers/view-zone.tsx`

Colocated under `app/drawers/view-zone/`:

| File | Role |
|------|------|
| `ViewZoneDialogs.tsx` | Quick-add, breakout, schedule dialogs |
| `ViewZoneSpeciesAndServicesSection.tsx` | Species / services / scheduled work |
| `ViewZoneEditForm.tsx` | Edit mode form (name, color, property, contains, etc.) |
| `ViewZoneDrawerChrome.tsx` | Header strip + breakout placement banner |

Supporting: `app/drawers/view-zone-constants.ts`.

### 4.4 AI chat — `app/drawers/ai-chat.tsx`

Colocated under `app/drawers/ai-chat/`:

| File | Role |
|------|------|
| `AiChatSidebar.tsx` | History / new chat |
| `AiChatThread.tsx` | Thread + suggestions; composes composer |
| `AiChatComposer.tsx` | Input / attachments |
| `GalleryPickerGrid.tsx` | Gallery grid |
| `AiChatGalleryPickerDialog.tsx` | My Photos dialog wrapper |
| `AiChatDeleteChatDialog.tsx` | Delete confirmation |

Supporting: `app/drawers/ai-chat-utils.ts`.

### 4.5 Profile — `app/drawers/profile.tsx`

Colocated under `app/drawers/profile/` (partial list; cards + collapsibles + `useProfileStripeActions.ts`):

| File | Role |
|------|------|
| `ProfileHeroCard.tsx`, `ProfilePlanBillingCard.tsx`, `ProfilePhotosStorageCard.tsx` | Major cards |
| `Profile*Collapsible.tsx` | Settings sections |
| `ProfileLogoutSection.tsx` | Log out control |
| `useProfileStripeActions.ts` | Stripe / billing actions |

---

## 5. Current line-count snapshot

From `npm run verify:drawers` (warns when **top-level** `app/drawers/*.tsx` **> 400 lines**). Snapshot taken at handoff authoring:

**Still over 400 lines (ordered by size):**

| Lines (approx.) | Entry file |
|----------------:|------------|
| 868 | `app/drawers/dashboard.tsx` |
| 843 | `app/drawers/client-form.tsx` |
| 778 | `app/drawers/view-zone.tsx` |
| 759 | `app/drawers/trees.tsx` |
| 752 | `app/drawers/team-settings.tsx` |
| 748 | `app/drawers/quote-form.tsx` |
| 692 | `app/drawers/invoice-form.tsx` |
| 671 | `app/drawers/job-form.tsx` |
| 659 | `app/drawers/hire-groundzy-pro.tsx` |
| 641 | `app/drawers/request-form.tsx` |
| 631 | `app/drawers/view-request.tsx` |
| 626 | `app/drawers/weather-impact.tsx` |
| 615 | `app/drawers/profile.tsx` |
| 606 | `app/drawers/contact-us.tsx` |
| 582 | `app/drawers/ai-chat.tsx` |
| 523 | `app/drawers/view-property.tsx` |
| … | (additional entries down to ~403) |

Re-run locally after changes:

```bash
npm run verify:drawers
```

---

## 6. Verification commands

| Command | Notes |
|---------|--------|
| `npm run verify:drawers` | Line-count warnings for top-level drawer entries. |
| `npx tsc --noEmit` | Typecheck; repo may still report unrelated test/vitest resolution issues. |

---

## 7. Smoke tests (after meaningful drawer splits)

Touch these paths manually when behavior could have shifted:

1. **AI chat:** send message, attachments, gallery picker.
2. **Request form:** assignees, submit create/update.
3. **Contact us:** inbox thread, **Contact Support** form, **gallery picker** (thread + support targets).

---

## 8. Explicitly out of scope (for this initiative)

- Broad **drawer `role` / RBAC** work — deferred until layout and entry-size discipline are stable (per original plans).

---

## 9. Suggested next steps for future work

1. Continue **Phase C** on remaining large entries: e.g. **`ai-chat.tsx`**, **`profile.tsx`**, then other top offenders from `verify:drawers` (`dashboard`, `client-form`, etc.) as scope allows.
2. Keep **`drawer-shell-classification.md`** updated whenever a drawer **root layout** changes (Shell vs documented exception).
3. Optionally refresh **`docs/drawer-audit.md`** §3 line counts if that doc is still the registry inventory you trust.
4. Prefer **one extraction per PR** with regression smoke on the three sensitive flows above.

---

## 10. Related documents

| Doc | Role |
|-----|------|
| [`docs/drawer-shell-classification.md`](drawer-shell-classification.md) | Shell vs exception rule + line budget note |
| [`docs/drawer-refactor-second-pass-retrospective.md`](drawer-refactor-second-pass-retrospective.md) | Detailed retrospective & inventories |
| [`docs/drawer-audit.md`](drawer-audit.md) | Registry / audit |
| [`docs/DRAWER_PR_CHECKLIST.md`](DRAWER_PR_CHECKLIST.md) | PR expectations |
| [`lib/drawers.ts`](../lib/drawers.ts) | Lazy registry (do not break public paths) |

---

*Handoff document generated for continuity after chat deletion. Amend when major drawer phases land.*
