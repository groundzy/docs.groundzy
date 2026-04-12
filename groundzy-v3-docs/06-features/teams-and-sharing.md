# Teams & sharing

## What it does

**Teams:** Org subscription, members, roles, invites (`invite_codes`), **team settings**, workflow defaults on team doc.  
**Sharing:** Share links (`/share/[token]`), grant by username, **tree permissions**, access requests, restricted tree view, bulk share.

## Who uses it

**Teams** tiers for team management; **sharing** for any user who owns or collaborates on trees.

## Data involved

- `teams/{teamId}`, `team_members`, `invite_codes`, `organizations/{orgId}` (rules).
- **User:** `organizationId`, `role`.
- **Sharing:** `tree_share_links`, `tree_permissions`, `user_tree_permissions`, `tree_access_requests` (`firebase/firestore.rules`).
- **Note:** Older internal docs mention `shared_data`; **rules** state Phase 2 deprecation in favor of **tree_permissions**—prefer rules as source of truth.

## UI patterns

`team-settings` drawer; view-tree **Share** tab; `restricted-tree` drawer; public route **`/share/[token]`**.

## Dependencies

- Auth, usernames collection
- API: share resolution, invite validate (`/api/invite-code/validate`, `/api/share/...`)

## Inconsistencies & overlaps

- **Team** id vs **organizationId** on user—two ways to refer to org membership.
- **Sharing** overlaps **trees** (permissions) and **workflow** only indirectly—no CRM link from share flow.
