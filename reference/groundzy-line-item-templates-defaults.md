# Groundzy line item templates: defaults and descriptions

**Source of truth (code):** `app/types/workflow-settings.ts` (in the `app` repo) — constant `GROUNDZY_DEFAULT_LINE_ITEM_TEMPLATES`

**Last reviewed with code:** 2026-04-22 (sync this date when the array in `workflow-settings.ts` changes)

---

## What these are

**Line item templates** are saved service line definitions for a team. They are stored in Firestore on the team as `workflowSettings.lineItemTemplates` (an array of `LineItemTemplate` objects). Teams use them to prefill quotes, invoices, and **request** flows (chips, template ids, and derived service types).

**Groundzy defaults** are the approved starter catalog shipped in app code. A team’s effective list is **Groundzy defaults** when:

- There is no team document, or
- `workflowSettings` is missing, or
- `lineItemTemplates` is empty or not a non-empty array

On the server, the same rule is implemented in `lib/groundzy/server/line-item-templates-from-team.ts` (`lineItemTemplatesFromTeamDocData`). The UI also offers **“Restore Groundzy defaults”** in team workflow settings, which **replaces** the stored list with a fresh copy of the array from code.

The catalog in this document is the **complete default text and metadata** for each Groundzy line item as defined in the TypeScript source. Teams may edit names and descriptions; restored defaults overwrite local edits.

---

## `LineItemTemplate` fields (relevant to defaults)

| Field | Purpose |
|--------|---------|
| `id` | Stable string id (e.g. `groundzy-tree-pruning`). Used for `lineItemTemplateIds` on requests and items. |
| `name` | Short label shown in UI and on documents. |
| `description` | Long default scope text; editable per team. |
| `type` | Line type for workflow line items: `labor` \| `material` \| `equipment` \| `disposal` \| `other`. All Groundzy defaults are **`labor`**. |
| `requestServiceType` | Request/quote “bucket” used for classifying the service: `pruning` \| `removal` \| `stump_grinding` \| `planting` \| `treatment` \| `assessment` \| `other`. Drives how requests derive **service type** from template ids. |
| `showOnRequestQuickBar` | If not `false`, the template appears as a **quick chip** on the request form; if `false`, it is only available from **More services** search. Omitted in older data is treated as **on** the quick bar. |

**Related product types (not in this file):** `RequestServiceType` in `app/types/request.ts`.

---

## Request form quick bar (defaults)

| Quick bar (chip row) | Search-only (long tail) |
|----------------------|-------------------------|
| Dormant Pruning, Tree Pruning, Stump Grinding, Tree Removal, Tree Planting | Crown Reduction, Hedge & Shrub Trimming, Plant Health Care, Soil & Fertilization, Tree Risk & Health Assessment, Arborist Consultation, General Tree & Landscape Service, Storm & Emergency Response |

`showOnRequestQuickBar: true` is set on the first five; `false` on the remaining eight. See `filterLineItemTemplatesForRequestQuickBar` in `app/lib/workflow/request-line-item-templates.ts`.

---

## Default catalog (all Groundzy line items)

The following subsections list **every** Groundzy default in **display order** as in `GROUNDZY_DEFAULT_LINE_ITEM_TEMPLATES`. Descriptions are **verbatim** from source.

### Dormant Pruning

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-dormant-pruning` |
| **type** | `labor` |
| **requestServiceType** | `pruning` |
| **showOnRequestQuickBar** | `true` |

**Description (default):**

> With Dormant Pruning the oldest branches are removed at ground level or from inside the plant to open it up and reduce size. Many shrubs flowering from new wood respond very well to Dormant Pruning, becoming healthier and producing vigorous blooms. Damage caused by shearing is corrected at this time as well.

---

### Tree Pruning

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-tree-pruning` |
| **type** | `labor` |
| **requestServiceType** | `pruning` |
| **showOnRequestQuickBar** | `true` |

**Description (default):**

> Remove deadwood, sucker growth, broken branches, and thinning out for air ventilation. Pruning and trimming of trees and shrubs will be done in accordance with the National Arborists' Association standards.

---

### Stump Grinding

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-stump-grinding` |
| **type** | `labor` |
| **requestServiceType** | `stump_grinding` |
| **showOnRequestQuickBar** | `true` |

**Description (default):**

> Professional stump grinding to remove tree stumps below grade. The stump is ground into mulch using specialized equipment, leaving the area level and suitable for replanting or landscaping.

---

### Tree Removal

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-tree-removal` |
| **type** | `labor` |
| **requestServiceType** | `removal` |
| **showOnRequestQuickBar** | `true` |

**Description (default):**

> Complete removal of tree from stump to crown. Includes felling, cutting into manageable sections, and removal of debris. Work performed in accordance with ANSI Z133 safety standards.

---

### Tree Planting

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-planting` |
| **type** | `labor` |
| **requestServiceType** | `planting` |
| **showOnRequestQuickBar** | `true` |

**Description (default):**

> Planting of new trees including site preparation, excavation, placement, backfilling, mulching, and initial watering. Species selection and placement follow best practices for long-term establishment.

---

### Crown Reduction

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-crown-reduction` |
| **type** | `labor` |
| **requestServiceType** | `pruning` |
| **showOnRequestQuickBar** | `false` |

**Description (default):**

> Targeted reduction of overall crown spread and/or height to reduce load on limbs, improve clearance, or manage risk, while maintaining tree form. Performed in accordance with ANSI A300 and industry best practices; never topping.

---

### Hedge & Shrub Trimming

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-hedge-shrub-trim` |
| **type** | `labor` |
| **requestServiceType** | `pruning` |
| **showOnRequestQuickBar** | `false` |

**Description (default):**

> Shaping, sizing, and renewal pruning for formal hedges, foundation plantings, and ornamental shrubs. Includes selective thinning where appropriate to maintain plant health and appearance.

---

### Plant Health Care (IPM)

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-plant-health-care` |
| **type** | `labor` |
| **requestServiceType** | `treatment` |
| **showOnRequestQuickBar** | `false` |

**Description (default):**

> Integrated pest and disease management: inspection, treatment plans, and follow-up. Uses targeted products and methods appropriate to the diagnosis, with emphasis on tree and environmental safety when applicable.

---

### Soil & Fertilization

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-soil-fertilization` |
| **type** | `labor` |
| **requestServiceType** | `treatment` |
| **showOnRequestQuickBar** | `false` |

**Description (default):**

> Soil health assessment, amendment recommendations, and deep-root or surface fertilization to support root development and vigor. Tailored to species, site conditions, and local regulations.

---

### Tree Risk & Health Assessment

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-tree-assessment` |
| **type** | `labor` |
| **requestServiceType** | `assessment` |
| **showOnRequestQuickBar** | `false` |

**Description (default):**

> Systematic visual assessment of tree structure, health, and site factors. Delivers a clear summary of conditions, risk context, and recommended next steps. Formal reports for hazard, pre-construction, or insurance available where offered.

---

### Arborist Consultation

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-consultation` |
| **type** | `labor` |
| **requestServiceType** | `assessment` |
| **showOnRequestQuickBar** | `false` |

**Description (default):**

> On-site or virtual walkthrough for planning, tree preservation, development impacts, or second opinions. Does not always include a full written report unless specified in scope.

---

### General Tree & Landscape Service

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-general-tree-care` |
| **type** | `labor` |
| **requestServiceType** | `other` |
| **showOnRequestQuickBar** | `false` |

**Description (default):**

> Custom scope of work to be defined on site or in the quote. Use when the job spans multiple service types or when the client’s needs do not map to a single standard line item.

---

### Storm & Emergency Response

| Attribute | Value |
|-----------|--------|
| **id** | `groundzy-storm-emergency` |
| **type** | `labor` |
| **requestServiceType** | `other` |
| **showOnRequestQuickBar** | `false` |

**Description (default):**

> Response to storm-damaged or failed trees: hazard mitigations, takedown of hanging limbs, and debris management as conditions allow. Safety and traffic/pedestrian clearance are prioritized.

---

## Summary table (at a glance)

| # | `id` | Name | `requestServiceType` | Quick bar |
|---|------|------|----------------------|-----------|
| 1 | `groundzy-dormant-pruning` | Dormant Pruning | pruning | yes |
| 2 | `groundzy-tree-pruning` | Tree Pruning | pruning | yes |
| 3 | `groundzy-stump-grinding` | Stump Grinding | stump_grinding | yes |
| 4 | `groundzy-tree-removal` | Tree Removal | removal | yes |
| 5 | `groundzy-planting` | Tree Planting | planting | yes |
| 6 | `groundzy-crown-reduction` | Crown Reduction | pruning | no |
| 7 | `groundzy-hedge-shrub-trim` | Hedge & Shrub Trimming | pruning | no |
| 8 | `groundzy-plant-health-care` | Plant Health Care (IPM) | treatment | no |
| 9 | `groundzy-soil-fertilization` | Soil & Fertilization | treatment | no |
| 10 | `groundzy-tree-assessment` | Tree Risk & Health Assessment | assessment | no |
| 11 | `groundzy-consultation` | Arborist Consultation | assessment | no |
| 12 | `groundzy-general-tree-care` | General Tree & Landscape Service | other | no |
| 13 | `groundzy-storm-emergency` | Storm & Emergency Response | other | no |

---

## Related documentation

- [Service type model spec](./service-type-model-spec.md) — broader `ServiceType` / catalog concepts (separate from `LineItemTemplate` ids)
- [Tree service taxonomy](./tree-service-taxonomy.md) — master list of services for modeling
- App workflow types: `app/types/workflow-settings.ts`, `app/lib/workflow/request-line-item-templates.ts`

---

## Changelog (doc maintenance)

- **2026-04-22** — Initial document: full Groundzy `GROUNDZY_DEFAULT_LINE_ITEM_TEMPLATES` copy, field semantics, quick bar split, Firestore/restore behavior. Update this file when the in-code default array or merge rules change.
