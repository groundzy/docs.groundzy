# Plant Health Care (PHC)

## What it does

**Planning / roadmap** feature (not fully described as shipped in the index table): structured **observations**, **treatment episodes**, **application lines**, programs, compliance, and links to **jobs** and tree history—see `docs/features/plant-health-care.md` for the authoritative plan.

## Who uses it

Intended for **team/pro** operational use in phases; exact tier gates are **product decisions** in the PHC doc.

## Data involved

- **Planned** extensions to existing types (`types/tree.ts`, `types/job.ts`, `zone-service`, etc.)—**not** a separate PHC-only collection summary in the short index; execution per PHC doc §locks.
- Reuses **tree history**, **workflow** line items, **zone services** conceptually.

## UI patterns

Doc prefers extending **view-tree**, **view-zone**, **view-job** rather than many new drawers—**avoid drawer sprawl**.

## Dependencies

- Trees, zones, workflow, map, AI (education), weather (snapshots)—cross-cutting.

## Inconsistencies & overlaps

- **PHC “service”** vs existing **`ServiceRecord`** in tree history—terminology split (episode vs service) per PHC doc.
- **Overlap** with **trees** history and **workflow** jobs—risk of **triple-logging** if not unified with **`Groundzy v3/05-data/event-system.md`** direction.
- **Status:** explicitly **planning**—do not assume production parity with the full PHC spec without code verification.
