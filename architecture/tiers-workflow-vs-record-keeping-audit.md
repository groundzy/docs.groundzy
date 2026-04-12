# Tiers: Teams workflow vs Home / Plus / Pro record-keeping — full repository audit

**See also:** [unified-tier-workflow-system-plan.md](./unified-tier-workflow-system-plan.md) — roadmap to merge into one system.

This document maps **every major place** the product splits **subscription tiers**, contrasts the **Teams business workflow** (requests → quotes → jobs → invoices, clients/properties) with **per-tree record keeping** (history entries, maintenance, service summaries), and lists **integration points** for a future **single unified system across all tiers**.

---

## 1. The conceptual split (what is “not one”)

| Axis | **Teams workflow** | **Home / Plus / Pro “record keeping”** |
|------|-------------------|----------------------------------------|
| **Purpose** | Business pipeline: clients, properties, operational documents (R/Q/J/I) | Tree-centric **history** (`tree.history.*`), **maintenance** blobs, **Add Record** / **entry-form** |
| **Primary data** | Firestore collections for requests, quotes, jobs, invoices; org-scoped | `ServiceRecord` / `HistoryEntry` on **tree** documents; optional `maintenance` |
| **Tier gate** | **`isTierInGroup(tier, "teams")`** — Small/Mid/Large/Enterprise Teams | **All tiers** can add trees; **Pro/Plus/Teams** differ on **clients/properties** and **zones** |
| **Navigation** | Sidebar **Workflow** group (`category: "workflow"` in `lib/drawers.ts`) only for Teams tiers | Map, trees, **entry-form**, **edit-tree**, add-tree flows for everyone in `HOME_PLUS_PRO_AND_TEAMS` |
| **UX hotspot** | View tree **Operations** tab: `WorkflowActiveWorkCard`; footer **Create workflow** vs **Add Record** | View tree **Activity** / history: `TreeHistoryView`; **Service Summary** + tags in **Operations** |

The two systems **meet** on a tree (e.g. workflow line items can reference `treeId` in forms), but **authorization and navigation** are implemented with **different hooks** and **different `visibleForTiers` arrays**.

---

## 2. Canonical tier utilities (single source of truth for “who am I?”)

| File | Role |
|------|------|
| `lib/utils/tier-utils.ts` | `getEffectiveSubscriptionTier`, `getUserSubscriptionTier`, `isTierInGroup` (groups: `home`, `plus`, `pro`, `teams`), `hasActivePaidSubscription`, `hasPaidTierIncomplete`, `normalizeTier` |
| `hooks/useTeamsOnlyAccess.ts` | **`hasAccess === isTierInGroup(tier, "teams")`** — used for **workflow UI** only |
| `hooks/useProOrTeamsAccess.ts` | **Pro + Plus + Teams** (and optional Home) for **clients/properties** surfaces |
| `hooks/useProOrTeamsAccess.ts` | **`useIsPlusOnlyProperties()`** — Plus-only **properties without clients** UX |

**Teams** in code = `Small Team`, `Mid Team`, `Large Team`, `Enterprise` (`tier-utils.ts`).

---

## 3. Drawer registry: where tiers diverge (`lib/drawers.ts`)

Constants:

- **`HOME_PLUS_PRO_AND_TEAMS`** — `['Home', 'Plus', 'Pro', Small/Mid/Large/Enterprise]`
- **`TEAMS_ONLY_TIERS`** — `['Small Team', 'Mid Team', 'Large Team', 'Enterprise']`

**Clients / properties (Pro + Teams, not Plus for sidebar “Clients” entry):**

- `clients-properties` — `visibleForTiers: ['Pro', ...Teams]` (not `Plus` in this list)
- `clients` duplicate — same restriction, `hideFromNav`

**Workflow pipeline (Teams only):**

- `requests`, `view-request`, `add-request`, `edit-request`
- `quotes`, `view-quote`, `add-quote`, `edit-quote`
- `jobs`, `view-job`, `add-job`, `edit-job`
- `invoices`, `view-invoice`, `add-invoice`, `edit-invoice`  

All use **`visibleForTiers: [...TEAMS_ONLY_TIERS]`** and many set **`category: "workflow"`**.

**Broadly available (includes Home/Plus):**

- `dashboard`, `weather`, `trees`, `view-tree`, `edit-tree`, `entry-form`, `search`, `profile`, etc. — **`HOME_PLUS_PRO_AND_TEAMS`** or explicit Home/Plus lists

**Consumed by** `lib/drawer-registry.ts` → `getSidebarDrawerMetadata`, `getBottomNavDrawerMetadata`, `getMoreDrawerMetadata` — **filters by `userTier`**.

---

## 4. `useTeamsOnlyAccess` — every call site (workflow / Teams)

| Location | Usage |
|----------|--------|
| `app/drawers/view-tree/index.tsx` | `hasTeamsAccess` → `ViewTreeContent` / footer |
| `app/drawers/view-tree/components/TreeOperationsTab.tsx` | `showUpgradePrompt` — Teams upgrade card vs full ops |
| `app/drawers/view-property.tsx` | `hasTeamsWorkflow` — footer buttons + workflow popover |
| `app/drawers/view-zone.tsx` | `hasTeamsWorkflow` — footer |
| `app/drawers/view-client.tsx` | `hasTeamsWorkflow` — footer |
| `app/drawers/components/WorkflowActiveWorkCard.tsx` | **Hides entire card** if not Teams |
| `app/drawers/request-form.tsx` | `showUpgradePrompt` |
| `app/drawers/quote-form.tsx` | `showUpgradePrompt` |
| `app/drawers/job-form.tsx` | `showUpgradePrompt` |
| `app/drawers/invoice-form.tsx` | `showUpgradePrompt` |
| `app/drawers/view-request.tsx` | `showUpgradePrompt` |
| `app/drawers/view-quote.tsx` | `showUpgradePrompt` |
| `app/drawers/view-job.tsx` | `showUpgradePrompt` |
| `app/drawers/view-invoice.tsx` | `showUpgradePrompt` |
| `app/drawers/requests.tsx` | `showUpgradePrompt` |
| `app/drawers/quotes.tsx` | `showUpgradePrompt` |
| `app/drawers/jobs.tsx` | `showUpgradePrompt` |
| `app/drawers/invoices.tsx` | `showUpgradePrompt` |
| `app/drawers/dashboard.tsx` | `hasTeamsAccess` — conditional sections |
| `hooks/useWorkflowSettings.ts` | Enables **team workflow settings** query only for Teams |

---

## 5. `useProOrTeamsAccess` / Plus — every call site (CRM / properties)

| Location | Notes |
|----------|--------|
| `app/drawers/view-property.tsx` | With `useTeamsOnlyAccess` — property drawer upgrade vs workflow |
| `app/drawers/view-client.tsx` | Same pattern |
| `app/drawers/view-tree/index.tsx` | `hasProOrTeamsAccess` for other gating |
| `app/drawers/property-form.tsx` | `showUpgradePrompt` + `useIsPlusOnlyProperties` |
| `app/drawers/client-form.tsx` | `showUpgradePrompt` |
| `app/drawers/clients-properties.tsx` | List drawer access |
| `app/drawers/search.tsx` | Search + Pro upgrade |
| `app/drawers/trees.tsx` | Tier checks inline with `isTierInGroup` |
| `app/drawers/profile.tsx` | Plan UI |
| `components/trees/add-tree-form.tsx` | `hasProOrTeamsAccess` for zone/pro features |

**Plus** is also referenced via **`usePlusCheckout`** in many drawers (upgrade flows): `dashboard`, `search`, `mapbox-map`, `view-zone`, `draw`, `upgrade-to-plus`, `multiple-add`, `groundzy-wizard`, `ai-identifying-wand`, `components/upgrade-limit-cta.tsx`, etc.

---

## 6. Navigation & URL guards (tier vs drawer metadata)

| File | Behavior |
|------|----------|
| `components/navigation/sidebar.tsx` | `getSidebarDrawerMetadata(userTier)` |
| `components/navigation/bottom-nav.tsx` | `getBottomNavDrawerMetadata` |
| `lib/i18n/navigation.ts` | Special-case labels for `trees` when `Home` / `Plus` |
| `components/work-area/work-area-content.tsx` | **`canUserAccessDrawer(drawerId, userTier)`** — blocks reopening tier-restricted drawers (e.g. deep link) |
| `components/work-area/work-area-desktop.tsx` | `userTier` for labels |
| `components/work-area/work-area-mobile.tsx` | `userTier` + `WORKFLOW_DRAWERS` dirty handling |

---

## 7. “Workflow” dirty-state set (cross-cutting; not tier-specific)

`stores/navigation-store.ts` — **`WORKFLOW_DRAWERS`** includes:

- `add-request` … `edit-invoice`, **`add-client`**, **`edit-client`**, **`add-property`**, **`edit-property`**, **`entry-form`**

So **record-keeping entry-form** shares the same “unsaved changes” plumbing as **CRM/workflow forms**.

---

## 8. Teams workflow — UI components (beyond drawers)

| File | Role |
|------|------|
| `app/drawers/components/WorkflowActiveWorkCard.tsx` | Active work tabs; **Teams-only** |
| `app/drawers/view-tree/components/TreeWorkflowSections.tsx` | View tree Operations: **only** `WorkflowActiveWorkCard` when property linked |
| `app/drawers/view-tree/components/TreeWorkflowCreateMenu.tsx` | Create R/Q/J/I from tree footer |
| `lib/workflow-pipeline-icons.ts` | Icons for workflow steps |
| `lib/workflow-step-icon-styles.ts` | CSS vars for workflow step icons |
| `lib/workflow-open-filters.ts` | “Open” / active rows for `WorkflowActiveWorkCard` |
| `components/settings/WorkflowSettingsEditor.tsx` | Team defaults (line items, tax, terms) |
| `hooks/useWorkflowSettings.ts` | Loads team workflow settings |
| `app/drawers/team-settings.tsx` | Embeds **`WorkflowSettingsEditor`** |
| `hooks/useWorkflowFormDirty.ts` | Syncs dirty state to store for **`WORKFLOW_DRAWERS`** |

---

## 9. Record keeping (tree history) — UI and data paths

| Area | Files / notes |
|------|----------------|
| **Tree history display** | `components/trees/tree-history-view.tsx`, `app/drawers/view-tree/components/TreeActivityTab.tsx` — **note: `hasTeamsAccess` prop is passed but unused in `TreeActivityTab`** (dead split) |
| **Add / edit entries** | `app/drawers/entry-form/index.tsx`, `components/trees/add-history-entry-form.tsx`, `useWorkflowFormDirty("entry-form", …)` |
| **Add tree** | `components/trees/add-tree-form.tsx` — large form; **tier** for zones/pro features; **history** embedded |
| **Tree document** | `types/tree.ts` — `history.serviceHistory`, `maintenance`, etc. |
| **Persistence** | `lib/firebase/firestore.ts` — push/update/delete service history, maintenance |
| **View tree Operations** | `TreeOperationsTab.tsx` — **Service Summary**, tags, **not** workflow lists (after recent simplification) |
| **View tree footer** | `ViewTreeActionFooter.tsx` — **`hasTeamsAccess`**: if **false**, primary = **Add Record** / schedule; if **true**, primary = **Create workflow** (hides Add Record / Schedule per comments) |

---

## 10. Map & dashboard

| File | Tier behavior |
|------|----------------|
| `components/map/map-toolbar.tsx` | **`teamsOnly: true`** on **Jobs** filter — only shown for **`isTierInGroup(tier, "teams")`**; `isProOrTeams` for other filters |
| `lib/utils/marker-zoom-size.ts` | Pro/Teams get different marker sizing |
| `app/drawers/dashboard.tsx` | `hasTeamsAccess` gates **jobs-this-week** / team-style blocks |

---

## 11. Billing / Stripe / API surface

| File | Role |
|------|------|
| `lib/stripe-webhook-handlers.ts` | `isTeams` tier branching |
| `app/api/stripe/create-checkout-session/route.ts` | Teams vs Pro checkout |
| `app/api/stripe/create-beta-checkout-session/route.ts` | Teams vs Pro |
| `app/drawers/profile/ProfilePlanBillingCard.tsx` | `isTierInGroup` for upgrade CTAs |

---

## 12. Copy & help

| Location | Content |
|----------|---------|
| `lib/i18n/messages.ts` | Many “available on Teams plans” / Pro+Teams strings |
| `app/drawers/help-faq-categories.ts` | Home vs Pro vs Teams FAQ |

---

## 13. Firestore rules (org vs home)

`firebase/firestore.rules` — **organization-scoped vs home-tier** (`organizationId == uid`) for trees/zones; **not** subscription-tier strings in rules (tier is enforced in **app** via `drawers.ts` + hooks).

---

## 14. Merge into one system — suggested workstreams

1. **Single “capabilities” module** — Derive from `userDoc` once: `{ canRecordHistory, canUseClientsProperties, canUseWorkflowPipeline, canUseTeamWorkflowSettings }` and replace ad-hoc `useTeamsOnlyAccess` + scattered `isTierInGroup` where possible.

2. **Unify navigation model** — Either expand **`visibleForTiers`** for workflow drawers to lower tiers with **feature flags**, or keep nav hidden but allow **URL + guard** to same components (today `work-area-content` partially does this).

3. **Data model** — Decide whether **non-Teams** users should get **lightweight** requests/jobs (e.g. personal-only) stored in the same collections with **different security rules**, or a **single “job-like” entity** that upgrades to full workflow when Teams.

4. **View tree** — Remove **`hasTeamsAccess` / `hasTeamsWorkflow` branching** in favor of one footer + one Operations model with **progressive disclosure** (record vs quote) instead of swapping primary CTA.

5. **Dead code cleanup** — `TreeActivityTab` `hasTeamsAccess` if unused; align **`WORKFLOW_DRAWERS`** naming vs “record” forms if product merges concepts.

6. **Docs** — Update `docs/features/current-workflow-audit.md` and any tree workflow plan files that still reference removed hooks.

---

## 15. File index (quick grep anchors)

- **Tier utils:** `lib/utils/tier-utils.ts`
- **Teams gate:** `hooks/useTeamsOnlyAccess.ts`
- **Pro/Plus gate:** `hooks/useProOrTeamsAccess.ts`
- **Drawer list:** `lib/drawers.ts`, `lib/drawer-registry.ts`
- **Navigation store / dirty:** `stores/navigation-store.ts`
- **Work area:** `components/work-area/work-area-content.tsx`
- **Workflow card:** `app/drawers/components/WorkflowActiveWorkCard.tsx`
- **Tree workflow UI:** `app/drawers/view-tree/components/TreeWorkflowSections.tsx`, `TreeWorkflowCreateMenu.tsx`, `ViewTreeActionFooter.tsx`
- **Team workflow settings:** `app/drawers/team-settings.tsx`, `hooks/useWorkflowSettings.ts`

---

*Generated by scanning the repo for tier hooks, `visibleForTiers`, workflow vs history paths. Update this file when merging tiers or changing `lib/drawers.ts`.*
