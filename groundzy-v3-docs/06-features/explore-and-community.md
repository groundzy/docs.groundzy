# Explore & community

## What it does

**Explore (`explore` drawer):** Species discovery / catalog browsing UI (`app/drawers/explore/` per registry).  
**Community:** Social feed–style features using `groundzy_posts`, `community_posts` (with likes/comments subcollections in `firebase/firestore.rules`)—exact product surface varies by implementation.

## Who uses it

Authenticated users where drawers are tier-visible; verify in `lib/drawers.ts` `visibleForTiers` for **`explore`**.

## Data involved

- `species_catalog` (explore-related queries)
- Community collections per rules; **tree** public/community linkage in **trees** / **tree_public** docs

## UI patterns

Explore uses **DrawerShell** per visual inventory; community UI may overlap **view-tree** Comments tab when present.

## Dependencies

- Species types, i18n
- Trees (public/community sharing settings)

## Inconsistencies & overlaps

- **Explore** vs **search** species results vs **tree-add** species autocomplete—**three** entry points to taxonomy.
- **Community** vs **hire-a-pro** vs **sharing**—different social graphs (feed vs pros vs token share).
