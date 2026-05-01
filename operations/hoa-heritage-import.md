# HOA / Heritage CSV import

Groundzy’s **admin** importer (`admin` app → Import Trees & Zones) ingests Heritage-style HOA inventory CSVs and writes **`trees`** and **`zones`** documents in the customer Firestore project.

## Column coverage

Expected columns include: `ID`, `Date`, `Common`, `Botanical`, `Height`, `DBH`, `Health`, `Tag ##`, `Qty`, `Objective`, `Address`, `Location`, `Hours`, `Price`, `Timing`, `Frequency`, `Description`, `Note`, `Hardscape`, `lat`, `lon`, `tagID`.

All columns are preserved on each write under **`importSource.rawRow`** for audit and re-import safety.

## Qty routing

| Condition | Document |
|-----------|----------|
| `Qty` blank, missing, or ≤ 1 | **Tree** |
| `Qty` > 1 | **Zone** (aggregate row) |

Zones get a **placeholder polygon** around the row’s lat/lon and **`needsBoundaryReview: true`**. Users adjust vertices in the **app** (View Zone → boundary editor).

## Idempotency

Each run computes **`importSource.fileId`** from the file name and content hash. Together with **`importSource.sourceRowId`** (inventory `ID`), duplicate detection skips rows that already exist when **Skip duplicates** is enabled.

The CSV **`ID`** is always preserved as `importSource.sourceRowId`. The admin importer also offers **Use CSV ID as treeNumber** for tree rows. When enabled, tree rows must have a positive whole-number `ID`; that value is written to `trees/{treeId}.treeNumber`. Zone rows (`Qty > 1`) keep using `ID` as their imported source/inventory ID, not as a tree number.

When CSV IDs are used as tree numbers, the importer performs a second collision check against existing non-deleted trees in the same `organizationId` + `databaseCode` scope. With **Skip duplicates** enabled, those rows are skipped; with it disabled, a tree-number collision is reported as an error. After successful writes, the importer advances `tree_number_counters/{scopeId}.nextNumber` to at least the highest imported tree number + 1 so future generated tree numbers do not collide with the imported inventory range.

For zone rows, the importer also checks for an existing non-deleted zone in the same `organizationId` + `databaseCode` scope with the same `importSource.sourceRowId`. This prevents duplicate aggregate zones when the same source inventory is re-exported or the CSV filename changes.

In the app map, when Labels are on and the label mode is **# Number**, trees show `treeNumber` and imported zones show `#` + `importSource.sourceRowId`. Manual zones without an imported source ID fall back to the zone name.

Composite indexes on Firestore:

- `trees`: `importSource.fileId`, `importSource.sourceRowId`, `isDeleted`
- `trees`: `organizationId`, `databaseCode`, `treeNumber`, `isDeleted`
- `zones`: `importSource.fileId`, `importSource.sourceRowId`, `isDeleted`
- `zones`: `organizationId`, `databaseCode`, `importSource.sourceRowId`, `isDeleted`

Deploy indexes with `firebase deploy --only firestore:indexes` from the **app** repo (canonical Firestore config).

## Measurements (Height / DBH)

Parsed values store:

- **`measurements.height.value`** / **`measurements.diameter.dbh`**: midpoint for sorting and legacy UI.
- **`range.min` / `range.max`**: when the CSV contained a range (e.g. `16'-30'`).
- **`rawText`**: original cell text.

The **app** displays ranges where present (e.g. `16.0–30.0 ft`) and falls back to midpoint-only strings otherwise.

## Objectives and estimates

Objectives are attached as **`workObjectives`** on the tree or zone (not as workflow line items at import time).

- **`Hours` > 0** → `estimate.estimatedHours`.
- **`Price` > 0** → `estimate.estimatedPrice` (USD).
- **`Price` = 0** → price treated as **unknown** (`priceStatus: "unknown"`), not “free”.
- **Zone rows (`Qty` > 1)** → `estimate.basis: "row_total"`, **`estimatedUnits`** = Qty, **`unitLabel`**: `trees` so hours are not interpreted as per-tree.

In the **app**, imported **hours/price** estimates are shown only for **Plus / Pro / Teams** viewers (`Recommended Work` card); raw objective text remains visible broadly.

## Visibility

- Imported **trees** default to **not public** (`isPublic: false`).
- **Zones** from import may carry **`importSource.needsReview`** metadata; boundary review is driven by **`needsBoundaryReview`**.

## Manual zone boundary review

After import, open the zone in the app → **Adjust boundary** → edit lat/lng vertices, add/remove points, save. Saving clears **`needsBoundaryReview`** and refreshes map layers.

## References

- Admin mapping: `admin/lib/importing/map-hoa-import.ts`
- Objective normalization: `admin/lib/importing/objective-normalize.ts`
- Measurement parsing: `admin/lib/importing/measurement-range.ts`
