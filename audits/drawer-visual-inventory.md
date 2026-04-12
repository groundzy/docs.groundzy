# Drawer visual inventory

**Purpose:** Per-drawer classification for shell layout, scroll behavior, card patterns, and notable visual overrides. Source of truth for IDs is [`getRegisteredDrawerIds()`](../lib/drawer-registry.ts) / [`lib/drawers.ts`](../lib/drawers.ts). Structural rules live in [`drawer-shell-classification.md`](drawer-shell-classification.md).

**Legend**

| Column | Values |
|--------|--------|
| **Shell** | `DrawerShell` \| exception (see classification doc) |
| **Scroll** | `DrawerScrollArea` \| `ListContainer` \| `ScrollArea` (Radix) \| embedded / custom |
| **Cards** | Primary patterns: `DrawerSection`, `contentCardClass` ([`lib/card-styles.ts`](../lib/card-styles.ts)), `listItemCardClass`, `brandedDeepTealCardClass`, bespoke |
| **Overrides** | e.g. `weather-drawer-gradient`, `bg-transparent` on shell, tier gates |

## Registry rows (one line per `registerDrawer` ID)

| ID | Entry / lazy import | Shell | Scroll | Cards | Overrides / notes |
|----|---------------------|-------|--------|-------|-------------------|
| dashboard | `app/drawers/dashboard` | DrawerShell | DrawerScrollArea | DrawerSection, list rows | Tea-green hover accents |
| weather | `app/drawers/weather-impact` | DrawerShell | DrawerBody + inner | DrawerSection | **`weather-drawer-gradient`** on scroll region |
| trees | `app/drawers/trees` | DrawerShell | **ListContainer** + filter | List / map items chrome | `bg-transparent` shell |
| tree-add | `app/drawers/tree-add` | **exception** | form-owned | AddTreeForm | Thin host |
| multiple-add | `app/drawers/multiple-add` | **exception** | embedded | DrawerBody / footer | `extendOverBottomNav` |
| search | `app/drawers/search` | **exception** | **ListContainer** | list rows | No DrawerShell |
| view-tree | `app/drawers/view-tree/index` | DrawerShell via layout | mixed | **contentCardClass**, tabs | ViewTreeDrawerLayout |
| entry-form | `app/drawers/entry-form` | DrawerShell | DrawerBody | form sections | — |
| restricted-tree | `app/drawers/restricted-tree` | DrawerShell | DrawerScrollArea | contentCardClass, DrawerSection | — |
| edit-tree | `app/drawers/edit-tree` | **exception** | form | AddTreeForm | — |
| view-zone | `app/drawers/view-zone` | DrawerShell | DrawerScrollArea | sections / cards | — |
| clients-properties | `app/drawers/clients-properties` | DrawerShell | **ListContainer** | list + filters | `bg-transparent` |
| clients | same as clients-properties | (same) | (same) | (same) | `hideFromNav` |
| view-client | `app/drawers/view-client` | DrawerShell | DrawerBody / scroll | DrawerSection | — |
| add-client / edit-client | `app/drawers/client-form` | DrawerShell | DrawerBody | **brandedDeepTealCardClass** blocks | Pro forms |
| properties | same as clients-properties | (same) | (same) | (same) | — |
| view-property | `app/drawers/view-property` | DrawerShell | DrawerBody | DrawerSection | — |
| add-property / edit-property | `app/drawers/property-form` | DrawerShell | DrawerBody | **brandedDeepTealCardClass** | — |
| requests | `app/drawers/requests` | DrawerShell | DrawerScrollArea | list | — |
| view-request | `app/drawers/view-request` | DrawerShell | DrawerBody | sections | — |
| add-request / edit-request | `app/drawers/request-form` | DrawerShell | DrawerScrollArea | form | Teams |
| quotes | `app/drawers/quotes` | DrawerShell | DrawerScrollArea | list | — |
| view-quote | `app/drawers/view-quote` | DrawerShell | DrawerBody | sections | — |
| add-quote / edit-quote | `app/drawers/quote-form` | DrawerShell | DrawerScrollArea | form | — |
| jobs | `app/drawers/jobs` | DrawerShell | DrawerScrollArea | list | — |
| view-job | `app/drawers/view-job` | DrawerShell | DrawerBody | sections | — |
| add-job / edit-job | `app/drawers/job-form` | DrawerShell | DrawerScrollArea | form | — |
| invoices | `app/drawers/invoices` | DrawerShell | DrawerScrollArea | list | — |
| view-invoice | `app/drawers/view-invoice` | DrawerShell | DrawerBody | sections | — |
| add-invoice / edit-invoice | `app/drawers/invoice-form` | DrawerShell | DrawerScrollArea | form | — |
| profile | `app/drawers/profile` | DrawerShell | DrawerScrollArea | sections / cards | — |
| public-profile | `app/drawers/public-profile` | DrawerShell | DrawerScrollArea | cards | — |
| my-photos | `app/drawers/my-photos` | DrawerShell | DrawerScrollArea | grid | — |
| hire-groundzy-pro | `app/drawers/hire-groundzy-pro` | DrawerShell | DrawerScrollArea | mixed | — |
| upgrade-to-plus / upgrade-to-pro / upgrade-to-teams | upgrade drawers | DrawerShell | DrawerScrollArea | marketing / **brandedDeepTealCardClass** | Gated branches `bg-background` |
| explore | `app/drawers/explore/index` | DrawerShell | embedded | SpeciesCard etc. | — |
| ai-identifying-wand | `app/drawers/ai-identifying-wand` | **exception** | wizard | embedded | Full-bleed wizard |
| ai-chat | `app/drawers/ai-chat` | DrawerShell | **ScrollArea** (thread) | chat UI | — |
| groundzy-wizard | `app/drawers/groundzy-wizard` | **exception** | custom tabs | embedded | Mobile wizard |
| tutorial | `app/drawers/tutorial` | DrawerShell | DrawerScrollArea | cards | — |
| help | `app/drawers/help` | DrawerShell | DrawerScrollArea | accordions / cards | — |
| contact-us | `app/drawers/contact-us` | DrawerShell | **ScrollArea** / DrawerScrollArea | inbox | Thread lists use Radix ScrollArea |
| team-settings | `app/drawers/team-settings` | DrawerShell | DrawerScrollArea | sections | — |
| draw | `app/drawers/draw` | DrawerShell | form | zone UI | `extendOverBottomNav` |
| measure | `app/drawers/measure` | DrawerShell | scroll | measure UI | — |

**Note:** The “more” drawer is synthetic (`drawer=more`) and not a separate `registerDrawer` ID.

## Related

- [`docs/features/drawer-system.md`](features/drawer-system.md) — visual consistency tiers and tokens
- [`lib/card-styles.ts`](../lib/card-styles.ts) — shared class constants
