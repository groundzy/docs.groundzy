# Add Request Form Drawer — Full Audit

**Scope:** The `add-request` drawer and its shared implementation with `edit-request` (`RequestForm`).  
**App paths:** `app/drawers/request-form.tsx`, `app/drawers/request-form/**`, `app/drawers/request-form-constants.ts`, `app/drawers/RequestFormMapMarkerThumb.tsx`.  
**Registry:** `lib/drawers.ts` (`add-request`, `edit-request`).  
**Audit date:** 2026-04-20.

---

## 1. Executive summary

The add-request experience is a **single large client component** (`RequestForm`) that powers both **create** (`add-request`) and **update** (`edit-request`) flows. Data entry is split into **card-based sections** with strong map integration (tree pick, center-on-property, show tree/zone on map). Creates go through the **Groundzy v3 event append API** (`workflow.request_created`); updates use **direct Firestore** mutation. Tier gating uses **`workflow.pipeline`** capability (Teams-style messaging in the upgrade UI). Overall behavior is coherent; main improvement themes are **i18n consistency**, **map-pick edge cases without a selected property**, **validation depth** (schedule sanity), and **continued decomposition** of the parent file for maintainability.

---

## 2. Registry and navigation

| Drawer ID       | Params (typed)                                                                 | Component   | Notes |
|-----------------|---------------------------------------------------------------------------------|-------------|-------|
| `add-request`   | `clientId?`, `propertyId?`, `contactId?`, `zoneId?`, `treeId?` (all optional) | `RequestForm` | `hideFromNav: true`, `extendOverBottomNav: true`, `archetype: "form"` |
| `edit-request`  | `requestId` (required)                                                          | `RequestForm` | Same shell; `effectiveDrawerId` switches dirty-tracking id |

**Entry points (non-exhaustive):**

- Requests list → `navigate("add-request")`
- View client / property / zone / tree workflows → `navigate("add-request", workflowParams)` with contextual ids
- `lib/workflow-nav-quick-add.ts` — quick-add path to `add-request`
- Sidebar label mapping (`components/navigation/sidebar.tsx`) for `"add-request"`

**Sub-flow navigation:** Add client / add property pass `returnTo: drawerId` so `navigation-store` allows hops without the discard dialog when `params.returnTo === current` drawer.

---

## 3. Architecture

### 3.1 Component tree

```
RequestForm (request-form.tsx)
├── DrawerShell
│   ├── form
│   │   ├── DrawerScrollArea
│   │   │   ├── [read-only banner if terminal status]
│   │   │   ├── RequestFormClientPropertySection
│   │   │   ├── RequestFormRequestDetailsSection
│   │   │   │     └── slot: RequestFormTreesAndZonesSection (conditional)
│   │   │   │           └── RequestFormItemServicesCollapsible (per selected tree/zone)
│   │   │   ├── RequestFormScheduleSection
│   │   │   ├── RequestFormAssigneesSection
│   │   │   └── RequestFormIntakeSection
│   │   └── DrawerFooter (Cancel / Create|Update)
│   └── workflowMapEntityContextMenus (portal content from hook)
└── [teams upgrade gate — early return]
```

### 3.2 State model

- **Local React state** for all fields (client, contact, property, title, description, intake, schedule, assignees, `requestItems`).
- **Dirty detection:** `pristineRef` holds a `JSON.stringify` snapshot; `formSnapshot` is recomputed each render and compared. Sorted arrays / normalized `requestItems` avoid ordering noise.
- **`useWorkflowFormDirty(effectiveDrawerId, isDirty)`** syncs to `navigation-store` when `activeDrawer` matches and drawer is in `WORKFLOW_DRAWERS` (`add-request` / `edit-request` included).
- **`beforeunload`** handler registered when dirty (browser tab close warning).

### 3.3 Edit vs add behavior

| Concern | Add | Edit |
|---------|-----|------|
| `workflowFormMode` | Always `"editable"` | From `getWorkflowFormModeFromRequestStatus` — terminal statuses → `readOnly` |
| Section policy | All sections editable | `getRequestFormSectionPolicy`: read-only locks all sections + `canSave: false` |
| Data load | Params + reset effect | `useRequest(requestId)` hydrates fields + pristine |
| Submit | `useCreateRequest` → `createRequestViaGroundzyEvent` | `useUpdateRequest` → `updateRequest` |
| Post-submit | `navigate("view-request", { requestId: newRequest.id })` | Same with existing id |

Terminal request statuses (read-only): `completed`, `converted`, `archived`, `declined`, `cancelled` (`workflow-form-mode-policy.ts`).

### 3.4 Hooks and services (dependency map)

| Area | Primary hooks / modules |
|------|-------------------------|
| Drawer / nav | `useDrawer`, `useNavigationStore` |
| Auth / org | `useAuth` (organizationId resolution chain) |
| CRM scope | `useClients`, `useClient`, `useProperties`, `useAutoSelectSingleProperty`, `filterPropertiesForWorkflowContact` |
| Request CRUD | `useRequest`, `useCreateRequest`, `useUpdateRequest` (`hooks/useRequests.ts`) |
| Team / tier | `useTeamsOnlyAccess` (`workflow.pipeline`), `useTeamAssignees` |
| Map | `useMapStore`, `useCenterMapOnProperty`, `useMapWorkflowTreePick`, `useWorkflowMapEntityContextMenus` |
| Trees / zones | `useTrees` (org-wide + property filter in memo), `useZones(propertyId)` |
| Species | `useSpeciesDisplayName`, `useSpeciesCatalog` |
| Dirty | `useWorkflowFormDirty` |
| Defaults | `getProSoloDefaultAssignedToUserIds` |

---

## 4. Data flow — create path

1. User fills `CreateRequestData`-shaped fields; client-side checks: `organizationId`, `clientId`, non-empty `title.trim()`.
2. `useCreateRequest` ensures Firebase auth + id token, calls `createRequestViaGroundzyEvent`.
3. Client generates **`requestId` = `crypto.randomUUID()`**; same value used as **idempotency key** and correlation id; POST to `/api/groundzy/events/append` with `type: "workflow.request_created"` and mapped payload (`mapCreateRequestDataToWorkflowPayload`).
4. On success, **`getRequest(requestId)`** loads projection; throws if not yet readable.
5. React Query invalidates `requests` list and participant workflow caches.

**Implication:** Create is **eventually consistent** with the projection; the code already handles “not yet readable” with a user-facing error.

---

## 5. UX and sections (behavioral notes)

### 5.1 Client & property

- Contact picker appears only for **business clients with contacts**.
- Changing contact **re-filters properties**; if current property not in filtered set, property cleared and **`requestItems` cleared**.
- Changing client clears contact, property, and request items.
- **Add client / add property** strip conflicting params and preserve `returnTo` for return navigation.

### 5.2 Request details

- **Title** auto-prefilled on add: `Request for ${getClientDisplayName(client)}` when `clientId` resolves; pristine snapshot patched if title was still empty (avoids false dirty after auto-title).
- **Service details** (request-level) and **per-tree/zone services** use the same option list as `SERVICE_DETAILS_OPTIONS` (constants), aligned with `RequestServiceType` in `types/request.ts`.
- **Trees & zones** block only renders when `propertyId` is set **and** there is at least one tree or zone on the property.

### 5.3 Schedule

- Date pickers store **ISO date strings `YYYY-MM-DD`** via `toISOString().slice(0, 10)` (watch for **timezone edge cases** around local midnight vs UTC for users far from UTC).
- **“Any time”** clears persisted start/end times on submit (undefined in payload).
- Changing **start time** auto-adjusts **end time** to +2 hours (modulo day); manual end time edits allowed afterward. No validation that end ≥ start or that end date aligns with date range.

### 5.4 Assignees

- Multi-select popover; options from `useTeamAssignees` plus any ids already selected (fallback label = raw id).
- **Pro solo default assignees** applied on add when list empty; second effect syncs pristine when defaults match expected ids (reduces dirty churn).

### 5.5 Intake

- Source / type / priority selects; **source/type option labels** in `request-form-constants.ts` are **English literals** (not `t()`), while section chrome uses i18n.

### 5.6 Map integration

- **Tree row hover** drives `map-store` `hoveredTreeId` (cleared on unmount).
- **Show on map** uses `requestCenterOn` with zoom **17** for trees, **16** for zones (first coordinate).
- **`useMapWorkflowTreePick`:** When map pick fires, if **no property** selected, hook still resolves tree by id and invokes callback with `{ hadNoProperty: true }`. **`RequestForm` ignores that context** and only calls `toggleTreeOnRequest(tree.id, true)` — it does **not** auto-select property/client from the tree. That may be intentional (minimal change) or a **product gap** for “pick tree first” flows.

---

## 6. Internationalization (i18n)

**Well covered:** Section titles, placeholders, trees/zones copy, schedule labels, most assignee strings (with fallbacks in `t()` for some keys), map error `requestForm.showOnMapNoLocation`.

**Gaps / inconsistencies:**

- **Upgrade panel:** “Service requests are available on Teams plans.”, “Upgrade to Teams” — hardcoded English.
- **Footer:** “Cancel”, “Saving…”, “Create”, “Update” — hardcoded English.
- **Toasts:** “Organization not found”, “Please select a client”, “Please enter a title”, “Request updated”, “Request created”, failure messages — hardcoded English.
- **`useMapWorkflowTreePick`:** “That tree is not on the selected property” — hardcoded English (triggered from this form’s usage).
- **`request-form-constants.ts`:** `REQUEST_SOURCES`, `REQUEST_TYPES`, `SERVICE_DETAILS_OPTIONS` labels are English-only; UI mixes translated section chrome with English enum labels in selects and toggles.
- **Assignees section** reuses `quoteForm.addClient` / `quoteForm.addProperty` keys indirectly only in client section; assignee-specific strings mostly use `requestForm.*`.

---

## 7. Accessibility and semantics

- Trees/zones use native **checkboxes** with associated `<label htmlFor=…>` — good.
- **Service detail toggles** use `<Button type="button" selected={…}>` — verify shared `Button` exposes correct `aria-pressed` / role (project pattern).
- **Assignee popover:** Custom checkbox visuals inside `<button>` — ensure focus order and `aria-expanded` on trigger (Radix Popover typically handles trigger).
- **Map marker thumb:** Container `aria-hidden` on wrapper but `img` has translated `alt` from `trees.requestFormMapMarkerAlt` — slight inconsistency (hidden from tree but alt present).

---

## 8. Performance and data fetching

- **`useTrees(organizationId)`** loads **all org trees**; form filters to `propertyId` in `useMemo`. Acceptable for small orgs; for large inventories consider **property-scoped query** if available.
- **`useSpeciesCatalog`** for marker icons and labels — global catalog; reasonable.
- **`speciesById` Map** rebuilt from catalog each render dependency — fine.
- **JSON.stringify dirty snapshot** on every render — simple but **O(n)** on large `requestItems`; unlikely problematic for typical counts.

---

## 9. Security and permissions

- **Create:** Server validates append payload (`/api/groundzy/events/append`); client sends Bearer token.
- **Update:** `updateRequest` in Firestore layer — rules must enforce org membership and status where applicable (out of scope for UI-only audit; confirm in rules tests).
- **Read-only edit UI** prevents submit when `!sectionPolicy.canSave`; still renders full form — ensure no bypass via keyboard on disabled button (browser default: disabled submit is good).

---

## 10. Testing

- **`lib/workflow/workflow-form-mode-policy.test.ts`** covers `getRequestFormSectionPolicy` / request status mode — good baseline for edit locking.
- **No dedicated tests found** for `request-form.tsx` (dirty snapshot, title auto-fill + pristine patch, contact/property clearing, map pick). High-value cases: pristine sync effects, `initialRequestItems` from `params.treeId` / `params.zoneId`, read-only submit guard.

---

## 11. Risks and issues (prioritized)

| Priority | Item |
|----------|------|
| P1 | **Map tree pick without property:** Callback ignores `hadNoProperty`; user may add a tree id without matching property context — confusing or invalid depending on backend rules. |
| P2 | **i18n:** User-visible English in upgrade CTA, footer, toasts, map toast, and constants-driven labels for enums. |
| P2 | **Schedule validation:** No checks for date order (start vs end) or time window sanity; timezone date string behavior. |
| P3 | **Loading:** Add flow has no explicit “loading clients/properties” skeleton; edit shows spinner when `isEdit && isLoading && !request`. |
| P3 | **`useEffect` on `[clientId, clients, isEdit]`** for auto-title: dependency list omits `params` / `pristineRef` patterns used elsewhere — works but is subtle; risk if client list updates reorder. |
| P4 | **Duplication:** `RequestFormAssigneesSection` also used from `job-form.tsx` — good reuse; parent `request-form.tsx` remains large (~740 lines). |

---

## 12. Recommendations

1. **Map pick:** When `useMapWorkflowTreePick` passes `hadNoProperty: true`, either **set `propertyId` / `clientId` from `tree.propertyId` / client resolution** or **toast** explaining that a property must be selected first — product decision, but avoid silent partial state.
2. **i18n sweep:** Move footer, upgrade panel, validation toasts, and `useMapWorkflowTreePick` message into `messages.ts` under `requestForm` or `workflow` namespaces; consider translating **constants labels** via keys (`value` stays typed; `labelKey` for `t()`).
3. **Schedule:** Add light validation (end date ≥ start date; if same day, end time ≥ start time unless `scheduleAnyTime`) and surface with `toast.error` or inline hints.
4. **Tests:** Add focused unit tests for dirty `pristineRef` + auto-title effect and integration test for create payload shape vs `CreateRequestData`.
5. **Optional refactor:** Extract “form state + snapshot” to `useRequestFormState` hook to shrink `request-form.tsx` and clarify effect ordering (document why title effect runs after reset effect).

---

## 13. File inventory

| File | Role |
|------|------|
| `app/drawers/request-form.tsx` | Main form, state, submit, map wiring |
| `app/drawers/request-form-constants.ts` | Zoom, zone helper, enum label lists |
| `app/drawers/RequestFormMapMarkerThumb.tsx` | Marker preview image |
| `app/drawers/request-form/RequestFormClientPropertySection.tsx` | Client, contact, property |
| `.../RequestFormRequestDetailsSection.tsx` | Title, service details, description, slot |
| `.../RequestFormTreesAndZonesSection.tsx` | Tree/zone selection + services + map actions |
| `.../RequestFormItemServicesCollapsible.tsx` | Per-item service toggles |
| `.../RequestFormScheduleSection.tsx` | Dates, any time, times |
| `.../RequestFormAssigneesSection.tsx` | Multi assignee popover |
| `.../RequestFormIntakeSection.tsx` | Source, type, priority, source details |
| `lib/drawers.ts` | Registration for `add-request` / `edit-request` |
| `lib/workflow/workflow-form-mode-policy.ts` | Section policy + terminal statuses |
| `lib/workflow/pro-solo-default-assignees.ts` | Default assignees (add flow) |
| `hooks/useRequests.ts` | Queries + mutations |
| `lib/groundzy/client/create-request-via-event.ts` | Create pipeline |
| `types/request.ts` | Types + `getRequestAssigneeUserIds` |

---

## 14. Conclusion

The add-request drawer is **feature-complete** for a workflow CRM: contextual entry from map/entity views, participant-aware list invalidation on create, terminal-status read-only editing, and integrated discard / `returnTo` behavior. The strongest follow-ups are **localization parity**, **map-pick behavior without a selected property**, and **lightweight schedule validation**, with optional structural refactor as the file grows.
