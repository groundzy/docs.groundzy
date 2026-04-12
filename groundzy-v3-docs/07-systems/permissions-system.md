# Permissions system (cross-feature)

**Authoritative detail (rules, roles, tree ACL, tier vs security):** [`../05-data/permissions.md`](../05-data/permissions.md) — including **execution order** (backend → org → tier → UI).

This file only captures **cross-cutting risks** and **gaps** that do not belong in the permissions reference.

---

## Why a second file exists

Permissions are implemented in **Firestore rules** + **client tier** + **tree ACLs**—not one module. The **anti-drift rule** is: **one** authoritative definition of *who can do what* in `permissions.md`; this file points there and records **system-level** concerns.

---

## Risks if layers drift

| Risk | Detail |
|------|--------|
| **Rules vs tier** | Developers assume **drawer visibility** = **data access**; rules must still **deny** unsafe writes. |
| **Two role stores** | `user.role` vs `teams.memberRoles` — see `permissions.md` §1. |
| **Scattered hooks** | `useProOrTeamsAccess`, `useTeamsOnlyAccess`, etc. — consolidate over time per [`../01-product/tier-system.md`](../01-product/tier-system.md) “Tier checks.” |
| **work_items in rules** | Helpers cap `treeIds` list length — unusual; see `permissions.md` §6. |

---

## v3 foundation direction

- **Policy** expressed as data + rules reviewed from one spec where feasible.
- **Tier** as server-resolved capability flags for sensitive ops—not only client drawers.
- **Single QA matrix** for tree ACL + org ACL ([`../v3-rebuild-alignment-plan.md`](../v3-rebuild-alignment-plan.md)).

---

## Related

- [`../05-data/permissions.md`](../05-data/permissions.md)
- [`../01-product/core-concepts.md`](../01-product/core-concepts.md) (Event + privacy)
