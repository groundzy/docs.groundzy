# Layout System (Groundzy v3)

Defines **spacing**, **page structure**, and **section hierarchy** so every surface aligns to the same rhythm. Values are expressed in **Tailwind** terms; v3 should centralize tokens (e.g. CSS variables or Tailwind theme extension) referenced by **Groundzy UI** only.

---

## 1. Spacing scale

Use a **restricted set** of spacing steps for padding and gaps (features apply via Groundzy components or layout tokens, not arbitrary `p-7` / `gap-3` unless token-mapped).

| Token | Typical use |
|-------|----------------|
| `xs` | Tight inline groups, icon gaps |
| `sm` | Between related controls in a field row |
| `md` | Default section padding, card body |
| `lg` | Between sections, drawer body padding |
| `xl` | Major section breaks, page gutters |

**Rule:** Feature code does **not** invent new spacing scales; Groundzy layouts embed the scale.

---

## 2. Page / drawer structure

Every drawer or full-page view follows the same **vertical stack**:

```
┌──────────────────────────────┐
│ Region: Header (optional)   │  ← title, breadcrumbs, primary actions
├──────────────────────────────┤
│ Region: Toolbar (optional)  │  ← filters, tabs, segment controls
├──────────────────────────────┤
│ Region: Body (scrollable)     │  ← primary content
├──────────────────────────────┤
│ Region: Footer (optional)   │  ← primary/secondary actions
└──────────────────────────────┘
```

- **Body** is the only scroll region by default (matches legacy `DrawerScrollArea` guidance in `docs/audits/drawer-shell-classification.md`).
- **Footer** stays outside scroll for sticky primary actions.

---

## 3. Section hierarchy

| Level | Usage |
|-------|--------|
| **Page / drawer title** | One H1-equivalent per surface (visually styled via `GzText` token, not raw `<h1>` in features if design system provides semantic wrapper). |
| **Section** | `GzDrawerSection` / `GzFormSection` — titled blocks with consistent `lg` padding and `md` gap between children. |
| **Subsection** | Optional subtitle or divider inside a section — same typography scale everywhere. |

**Rule:** No **ad hoc** `Card` stacks for “sections” in features — use Groundzy section primitives.

---

## 4. Grid and width

- **Drawer content:** max-width constraints applied in **Groundzy** drawer body (e.g. `max-w-prose` for long copy) — not per-feature `max-w-*` on random wrappers.
- **Two-column** layouts (e.g. form + preview) use **`GzSplitLayout`** (or equivalent) with tokenized breakpoints.

---

## 5. Elevation and borders

- **Cards** use one **border + shadow** token set (`GzCard`).
- **Workflow / semantic accents** (e.g. pipeline step colors) use **CSS variables** from product tokens (see workflow color rules in repo) — not arbitrary `border-l-4` per drawer.

---

## 6. Responsive behavior

- **Mobile-first:** Drawer footers and bottom nav follow **Groundzy** patterns; features do not add bespoke `fixed` chrome.
- **Touch:** Minimum hit targets (e.g. 44px) enforced in **GzButton**, **GzNavItem**.

---

## Related

- [`component-standards.md`](./component-standards.md) — which components implement this system
- [`interaction-patterns.md`](./interaction-patterns.md) — motion and state presentation
