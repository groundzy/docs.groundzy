# Collaborator & org role — UX guidelines

**Audience:** Product, design, frontend, support.

**Normative architecture:** [`unified-permission-model-v3.md`](../groundzy-v3-docs/05-data/unified-permission-model-v3.md).

---

## Problem

Users have **two axes** that interact:

1. **Organization role** (`TeamRole` on the team — viewer through owner).
2. **Tree collaborator role** (`tree_permissions/{treeId}/members/{uid}.role` — grant roles such as viewer, contributor, editor, manager, owner).

Those **compose** in policy ([`canOrgOrTreeAction`](../../app/lib/groundzy/policy/tree-collaborator.ts)); they are **not** two unrelated products. Users will still perceive “two roles” unless the UI labels them clearly.

---

## Rules

1. When showing team membership, always prefix with context: **Org:** or **Organization:** followed by the `TeamRole` (e.g. **Org: Viewer**).
2. When the user has an explicit tree grant (shared tree), show **On this tree:** followed by the grant role (e.g. **On this tree: Editor**).
3. **Never** display only **“Viewer”** or **“Member”** without clarifying whether that is **org** or **on this tree** when both apply.
4. Prefer a **two-line** or **compound** badge in tree headers, share panels, and permission administration screens.
5. Support macros: explain that **Org: Viewer** can still **edit this tree** if **On this tree: Editor** — collaborator grant overrides read-only org default **for that tree**.

---

## Related

- [`role-capabilities-matrix.md`](./role-capabilities-matrix.md)
- [`teams-and-roles-overview.md`](./teams-and-roles-overview.md)
