# CRM (clients & properties)

## What it does

**Clients:** contact and company records for people you serve.  
**Properties:** site/address records (and boundaries) linked to clients; anchor for trees and workflow.

## Who uses it

**Pro** and **Teams** for full clients/properties surfaces (`docs/features/crm.md`, `useProOrTeamsAccess`). **Plus** may have **limited** property behavior (`useIsPlusOnlyProperties` in codebase—verify UI).

**Home** does not use CRM lists.

## Data involved

- `clients/{clientId}`, `properties/{propertyId}` (`types/client.ts`, `types/property.ts`).
- Scoped by **`organizationId`** (team) or user **`databaseCode`** patterns (see CRM doc).

## UI patterns

List + **view** + **add/edit** drawers (`clients-properties` combined list, or separate `clients` / `properties` hidden nav entries). **Map:** property polygons (`property-layer.tsx`). **Branded** form cards on Pro forms (`docs/audits/drawer-visual-inventory.md`).

## Dependencies

- Map (property layer), tier hooks
- Workflow entities reference **clientId** / **propertyId** (`workflow.md`)

## Inconsistencies & overlaps

- **CRM doc** historically bundled **workflow** entities in one file; this repo splits **workflow.md** for clarity—**same data**, different life cycle.
- **Property** optional `clientId` vs **tree** optional `clientId`/`propertyId`—multiple ways to link the same real-world relationship.
- **Plus vs Pro** property gating—two mechanisms (`Plus` drawer visibility vs **Pro** CRM)—easy to confuse in docs.
