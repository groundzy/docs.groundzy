# UI Principles (Groundzy v3)

## Purpose

Groundzy v3 delivers **one cohesive product**: one visual language, one interaction vocabulary, one path from primitives to screens. Users should not feel they are switching between unrelated mini-apps when moving from map to CRM to workflow.

---

## Stack

| Layer | Role |
|-------|------|
| **Tailwind CSS** | Tokens, layout, responsive behavior — consumed by Groundzy UI and global styles, not sprinkled arbitrarily in features. |
| **Radix** | Accessibility and behavior primitives — **not imported in feature code**; exposed only through Groundzy UI or shadcn wrappers inside the Groundzy UI package. |
| **shadcn/ui** | Low-level, themeable components — **implementation detail** behind Groundzy UI; **never** imported in feature modules. |

---

## Principles

### 1. Groundzy UI is the only product-facing UI API

Features compose **Gz*** components. If something is missing, **extend Groundzy UI** first—not the feature.

### 2. Consistency over novelty

New screens use **existing archetypes** (list, detail, form, wizard). Variation is expressed through **content and density**, not new chrome.

### 3. Accessibility is non-negotiable

Focus order, labels, keyboard paths, and ARIA live on **Groundzy** components so every feature inherits them. Radix’s strengths are **centralized**, not re-implemented per screen.

### 4. Density for field use

Tree care users work outdoors and on tablets. Touch targets, contrast, and scanability take precedence over decorative minimalism.

### 5. Internationalization at the edge

User-visible strings flow through the same i18n layer as today; Groundzy UI components accept **keys or render props** for labels, not hardcoded English inside primitives.

### 6. Theming and brand

Brand colors and semantic tokens (e.g. workflow step colors) live in **design tokens** referenced by Groundzy UI—features do not apply ad hoc hex or arbitrary Tailwind classes for the same semantic.

---

## Anti-patterns (legacy to avoid)

- Importing `Button`, `Card`, `Dialog` from `@/components/ui` inside `app/drawers/*` or feature folders.
- **Exception drawers** without a shared shell (see `docs/drawer-shell-classification.md`) — in v3, exceptions are **rare** and implemented **inside** Groundzy UI (e.g. `GzWizardLayout`), not in raw feature JSX.

---

## Related

- [`design-system.md`](./design-system.md) — layers and audit
- [`component-standards.md`](./component-standards.md) — concrete rules
- [`layout-system.md`](./layout-system.md) — spacing and hierarchy
