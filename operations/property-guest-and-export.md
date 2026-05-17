# Property guest hub links and inventory export (CSV / PDF)

Operational reference for **property-scoped client hub** access and **property inventory** downloads (Pro/Teams, staff-only).

## Feature summary

- **Guest link:** Staff mints an opaque token in `property_guest_tokens`. The client opens `/property-guest/{token}`; the app sets the usual hub cookie with a **property scope** so `/client-hub` only lists documents and trees for that property.
- **CSV export:** Staff downloads `GET /api/properties/{propertyId}/export-inventory` (or `?format=csv`) with a Firebase ID token. Response is UTF-8 CSV with BOM (Excel-friendly), capped tree count and history rows (see app `lib/property-export/build-property-inventory-csv.ts` and shared loader `lib/property-export/load-property-trees-export.ts`).
- **PDF report:** Same route with `?format=pdf` returns `application/pdf` — cover, three summary metrics (trees, health-assessed count, zone count for the property), **site map** (static Mapbox image or on-page placeholder copy if unavailable), then **one section per tree** (species marker, measurements, location, health notes, activity) — no duplicate inventory table. Implementation: `lib/property-export/render-property-report-pdf.ts`, context `lib/property-export/property-report-data.ts`. Content caps match CSV via `loadPropertyTreesForExport`.
- **Subset export:** Optional explicit tree IDs limit the CSV/PDF to those trees only (same auth and gates). **`GET`** supports repeated query params `treeIds=<id>` (comma-separated values in a single param are also split). **`POST`** `/api/properties/{propertyId}/export-inventory` with JSON `{ "format": "csv" | "pdf", "treeIds": string[] }` is required when selecting **more than 50** trees (URL length); the app uses **`EXPORT_INVENTORY_MAX_TREE_IDS_GET`** (see `lib/property-export/export-inventory-constants.ts`). Every ID must exist and belong to the route’s `propertyId` and org; otherwise the API returns **400** (`tree_id_not_found`, `tree_id_not_in_scope`, etc.). Max **500** distinct IDs per request (`PROPERTY_EXPORT_MAX_TREE_IDS_REQUEST`). Output filenames use a `selected` segment when exporting a subset. PDF copy distinguishes **full property** vs **selected trees** (metrics and zone list follow the subset when applicable).

## Map snapshot (PDF)

- Prefer **`MAPBOX_ACCESS_TOKEN`** (server-only secret); otherwise **`NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN`**. Used for Mapbox Static Images (`lib/property-export/mapbox-static-property.ts`).
- Tree pins are capped (see `PROPERTY_REPORT_MAP_MAX_PINS`); the PDF can print how many pins were shown vs trees with coordinates.
- If the token is missing, the center cannot be computed, or the HTTP request fails, the PDF still shows a **Site map** section with a short explanation (not a silent gap). Check server logs for `[property-map-snapshot]` warnings on failure.

## Groundzy branding, QR codes, and hub URLs (PDF)

- Closing section includes **marketing / auth / app** URLs and optional **QR** tiles.
- **`NEXT_PUBLIC_GROUNDZY_MARKETING_URL`** — defaults to `https://groundzy.com`.
- **`NEXT_PUBLIC_GROUNDZY_AUTH_URL`** — defaults to `https://auth.groundzy.com`.
- **`NEXT_PUBLIC_APP_URL`** — used with `getPortalBaseUrl()` for staff app and hub paths (see `lib/documents/mint-portal-token-server.ts`).
- **`NEXT_PUBLIC_SUPPORT_EMAIL`** — optional support line on the PDF.
- **Property guest URL in PDF (optional):** Set **`GROUNDZY_PROPERTY_PDF_INCLUDE_GUEST_URL=1`** to embed a **single active** `property_guest_tokens` link (longest unexpired) for that property **when one exists**. The PDF then acts like a **bearer** for that hub scope until token expiry — treat printed or forwarded PDFs like shared passwords. If unset, guest URLs and the second QR are omitted.

## Requirements

- Property must have a **CRM `clientId`** to mint a guest link (product rule).
- **Subscription:** Same tier gate as Connect client payments (Pro, Small/Mid/Large Team, Enterprise) — see `assertUidTierAllowsPropertyExportFeatures` in the app repo.
- **Staff access:** `verifyStaffPortalMintAccess` for the property’s `organizationId`.

## Kill switch

Set `GROUNDZY_DISABLE_PROPERTY_GUEST=1` to return 503 on mint, session-from-property-token, and export routes.

## Rate limits (in-process)

Kinds: `property_guest_mint`, `property_guest_session`, `property_export` — see `lib/server/portal-rate-limit.ts` and env vars `PORTAL_RL_PROPERTY_*`.

## Logs

Structured events: `property_guest_mint`, `property_guest_session_from_token`, `property_export_csv`, `property_export_pdf` (prefix `[document-portal]`). Token values are not logged; token suffix may appear for correlation. Export logs may include `subset: true`, `treeCount`, and `requestedIdsCount` (subset flows).

## Revoking access

Set `status: "revoked"` on the `property_guest_tokens/{token}` document in Firestore. Existing hub cookies remain valid until expiry unless you rotate `CLIENT_HUB_SESSION_SECRET` (invalidates all hub sessions).

## PII / exports

Exports include tree locations, species, health, and history text. Treat files as **sensitive**; share only with the intended client.
