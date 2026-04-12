# Groundzy Design System (v3)

This document defines the **unified design system** for Groundzy v3: layers, scope, relationship to the current stack, and how we eliminate “mini-app per feature” UI.

**Stack retained (primitives only):** Tailwind CSS, Radix UI, shadcn/ui — wired through a **Groundzy UI** layer. See [`ui-principles.md`](./ui-principles.md).

---

## 1. Audit: what the legacy codebase shows

Evidence from internal docs and spot-checks of `app/drawers/**`:

| Finding | Source / example |
|--------|-------------------|
| **Many drawer shell patterns** | `docs/audits/drawer-visual-inventory.md` — mix of `DrawerShell`, `ListContainer`, `ScrollArea`, exceptions without shell (`search`, `tree-add`, wizards). |
| **Documented exceptions** | `docs/audits/drawer-shell-classification.md` — multiple approved non-standard roots (search, tree-add, identifying wand, groundzy-wizard, etc.). |
| **Per-drawer visual overrides** | Weather gradient, `bg-transparent` shells, `brandedDeepTealCardClass`, `contentCardClass` — same product, many visual dialects. |
| **Direct shadcn in feature code** | e.g. `app/drawers/job-form.tsx`, `client-form.tsx`, `view-tree/index.tsx` import `Button`, `Card`, `Dialog` from `@/components/ui/*` directly. |
| **Modals / dialogs** | Mix of `Dialog` from ui, `ConfirmDestructiveDialog`, zone-specific dialog components — not one composable family in feature land. |
| **Forms** | Shared primitives exist, but features assemble labels, inputs, cards per screen with duplicated layout patterns. |
| **Maintainability** | `docs/audits/drawer-system-current-state.md` — megafiles, consistency score below architecture score. |

**Conclusion:** The legacy app already has **drawer-layout** and **ui** building blocks, but **features bypass a strict product layer** and import shadcn directly. v3 **formalizes** that layer and **enforces** it.

---

## 2. Core UI layers (v3)

```
┌─────────────────────────────────────────────────────────────┐
│  Feature modules (routes, drawers, domain containers)      │
│  — NO @/components/ui/* — NO raw Radix in feature folders    │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│  Groundzy UI (@groundzy/ui or @/components/groundzy-ui/...)   │
│  — GzButton, GzDrawer, GzModal, GzForm*, GzEntityShell, …    │
│  — composes tokens, density, a11y, product copy slots        │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│  Primitives (shadcn/ui) — internal to Groundzy UI only       │
│  Button, Card, Dialog, Input, … — single import surface      │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│  Radix + Tailwind — implementation detail of primitives      │
└─────────────────────────────────────────────────────────────┘
```

**Aliases:** Exact package path is a repo decision; the rule is **feature → Groundzy UI only**.

---

## 3. Standardized surfaces (one system each)

| Surface | Groundzy UI owns | Primitive basis |
|---------|------------------|-----------------|
| Drawers | `GzDrawer*` shell, scroll regions, footer | shadcn layout + existing drawer-layout concepts |
| Modals | `GzModal`, `GzConfirmDialog` | Dialog / AlertDialog |
| Forms | `GzForm`, `GzFormField`, `GzFormSection` | RHF + Zod at feature boundary; primitives inside Gz |
| Lists / data | `GzDataList`, `GzEntityCard` | Card, table primitives |
| Entity views | `GzEntityShell` (header, tabs, actions) | Composed from Gz components |
| Navigation | `GzNav`, sidebar patterns | Nav primitives |

Details: [`component-standards.md`](./component-standards.md).

---

## 4. Rules (enforceable)

See [`component-standards.md`](./component-standards.md) § Rules. Summary:

- Features **must not** import `@/components/ui/*` or `@radix-ui/*`.
- Features **must not** implement one-off drawer roots; use **Groundzy drawer shells** only.
- **No duplicated** layout patterns for the same archetype (list vs detail vs form).

**Enforcement (recommended):** ESLint `no-restricted-imports` on `app/**`, `features/**`; allowlist only Groundzy UI + domain hooks; CI fails on violation.

---

## 5. Related documents

| Document | Contents |
|----------|----------|
| [`ui-principles.md`](./ui-principles.md) | Principles, accessibility, density |
| [`component-standards.md`](./component-standards.md) | Component catalog, rules, mapping |
| [`layout-system.md`](./layout-system.md) | Spacing, grid, section hierarchy |
| [`interaction-patterns.md`](./interaction-patterns.md) | Loading, empty, error, motion, nav |
| [`../03-architecture/routing-and-navigation.md`](../03-architecture/routing-and-navigation.md) | Drawers as **system primitive** (URL + shell); `GzDrawer` aligns here |

---

## 6. Alignment with product foundation

[`Groundzy v3/00-foundation/principles.md`](../00-foundation/principles.md) requires **consistent UI/UX** and **system-first** thinking. This design system is the **implementation** of that requirement for the frontend.
