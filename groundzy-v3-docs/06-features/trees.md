# Trees (map items)

## What it does

Full lifecycle for **tree records**: add (single, multiple, from zone), view/edit, **history** entries (service, inspection, note, measurement), media, sharing, archive, species catalog & **Quick Picks**, AI identify handoff, community/public options.

## Who uses it

All tiers; **Home** tree limits per **feature README** (5)—**Plus/Pro** unlimited in that matrix; verify against product copy. **CRM links** (property/client) more relevant for **Pro+**.

## Data involved

- **Primary:** Firestore `trees/{treeId}` (`types/tree.ts`).
- **History:** Embedded `history` + `tree_events/{treeId}/events` + dual-write **`work_items`** when enabled (`lib/firebase/firestore.ts`, `work-items.ts`).
- **Media:** Storage paths under org/tree (`docs/features/trees.md`).
- **Species:** `species_catalog`, quick picks collections; `GET /api/quick-picks`.

## UI patterns

Drawers: `trees`, `tree-add`, `view-tree`, `edit-tree`, `entry-form`, `multiple-add`, `restricted-tree`. Large **`add-tree-form`** orchestration. **Tabs** on view-tree (overview, activity, ops, … per current app).

## Dependencies

- Map, selection store, tree usage hooks
- Optional: property, client, zone, AI identify store
- i18n, tier gates

## Inconsistencies & overlaps

- **History triple path:** embedded `TreeHistory`, `tree_events`, and **work_items** mirrors—see `Groundzy v3/05-data/event-system.md`.
- **“View-tree” tabs** naming vs **Activity** merge flag (`UserPreferences`)—same screen, different data sources merged in UI.
- **CRM** overlaps: `propertyId`, `clientId` on tree links trees to **crm.md** entities.
