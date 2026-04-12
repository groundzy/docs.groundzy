# Hire a Pro

## What it does

Helps users **find and contact** nearby tree care professionals: **Groundzy Certified Pros** (curated Firestore list) and **Google Places** results; contact creates **`pro_contact_requests`**; optional default pro on user doc.

## Who uses it

**Home** and **Plus** only—drawer **hidden** for **Pro/Teams** (`docs/features/hire-pro.md`) because those users are treated as **service providers**, not consumers of this flow.

## Data involved

- `groundzy_pros` (admin-managed reads via API)
- `pro_contact_requests` (user-created)
- User doc fields for default pro
- **Places:** external API via `/api/places/nearby-tree-services`

## UI patterns

`hire-groundzy-pro` drawer; multi-step intro; map center / GPS for location (overlaps **map.md**).

## Dependencies

- Map location, geocode
- Env: Places/Maps keys

## Inconsistencies & overlaps

- **Parallel “find a pro”** story vs **CRM Clients**—different problems (B2C discovery vs B2B CRM).
- **“Pro”** in product name vs **Pro tier**—word collision for documentation.
