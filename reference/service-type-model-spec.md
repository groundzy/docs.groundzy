# ServiceType model spec

**ServiceType** should stay **clean and stable**: a **catalog definition**, not a runtime instance. Taxonomy, targeting rules, billing defaults, execution behavior, and history behavior live here—not in tree history or CRM documents.

**Related docs**

- [Work item system architecture](../architecture/work-item-system-architecture.md) — WorkItem as system of record, CRM linking, Firestore, migration
- [Tree service taxonomy](./tree-service-taxonomy.md) — human-readable master list of services by category

---

## Why this shape is “clean”

It separates:

- taxonomy
- targeting rules
- billing defaults
- execution behavior
- history behavior

That lets the UI:

- show only services valid for a **zone**
- auto-classify an inspection as an **inspection** history output
- default `tree_removal` to urgent / high-risk workflows
- allow `seasonal_inspection` (and similar) to be **recurring**
- avoid odd billing (e.g. “tree valuation” as per-stump)

---

## Type definitions

Proposed location: `types/service-types.ts`.

```ts
export const SERVICE_CATEGORIES = [
  "planting_establishment",
  "watering_irrigation",
  "pruning_trimming",
  "risk_safety",
  "removal_stump",
  "pest_disease",
  "soil_root",
  "storm_emergency",
  "inspection_assessment",
  "construction_protection",
  "growth_management",
  "specialty_utility",
  "maintenance_recurring",
  "documentation_reporting",
  "consulting_advisory",
] as const;

export type ServiceCategory = typeof SERVICE_CATEGORIES[number];

export const SERVICE_TARGET_TYPES = [
  "tree",
  "zone",
  "property",
  "client",
  "site",
  "multi_tree",
] as const;

export type ServiceTargetType = typeof SERVICE_TARGET_TYPES[number];

export const SERVICE_EXECUTION_MODES = [
  "one_time",
  "recurring",
  "inspection",
  "monitoring",
  "emergency",
] as const;

export type ServiceExecutionMode = typeof SERVICE_EXECUTION_MODES[number];

export const SERVICE_BILLING_MODES = [
  "fixed",
  "time_and_material",
  "unit",
  "per_tree",
  "per_zone",
  "per_visit",
  "inspection_fee",
  "quote_only",
  "non_billable",
] as const;

export type ServiceBillingMode = typeof SERVICE_BILLING_MODES[number];

export const SERVICE_PRIORITY_DEFAULTS = [
  "low",
  "normal",
  "high",
  "urgent",
  "emergency",
] as const;

export type ServicePriorityDefault = typeof SERVICE_PRIORITY_DEFAULTS[number];

export type ServiceTypeId =
  | "tree_planting"
  | "tree_transplanting"
  | "replacement_planting"
  | "staking_guying"
  | "soil_preparation"
  | "watering"
  | "deep_root_watering"
  | "irrigation_installation"
  | "irrigation_maintenance"
  | "structural_pruning"
  | "crown_cleaning"
  | "crown_thinning"
  | "crown_raising"
  | "crown_reduction"
  | "directional_pruning"
  | "clearance_pruning"
  | "vista_pruning"
  | "pollarding"
  | "deadwood_removal"
  | "hazard_assessment"
  | "cabling"
  | "bracing"
  | "lightning_protection"
  | "tree_removal"
  | "sectional_dismantling"
  | "crane_removal"
  | "emergency_removal"
  | "stump_grinding"
  | "stump_removal"
  | "root_removal"
  | "debris_hauling"
  | "site_cleanup"
  | "pest_inspection"
  | "disease_diagnosis"
  | "ipm"
  | "insecticide_treatment"
  | "fungicide_treatment"
  | "soil_drench"
  | "trunk_injection"
  | "soil_testing"
  | "soil_amendment"
  | "fertilization"
  | "compost_application"
  | "aeration"
  | "vertical_mulching"
  | "root_collar_excavation"
  | "decompaction"
  | "biochar_application"
  | "mycorrhizal_inoculation"
  | "storm_damage_assessment"
  | "fallen_tree_removal"
  | "hanging_limb_removal"
  | "post_storm_cleanup"
  | "tree_health_assessment"
  | "risk_assessment"
  | "pre_construction_assessment"
  | "annual_inspection"
  | "inventory_tagging"
  | "monitoring_plan"
  | "decay_detection"
  | "root_inspection"
  | "tree_protection_planning"
  | "tpz_setup"
  | "construction_monitoring"
  | "root_pruning"
  | "barrier_installation"
  | "compliance_reporting"
  | "growth_regulator_application"
  | "canopy_training"
  | "structural_correction"
  | "fruit_tree_pruning"
  | "orchard_management"
  | "utility_line_clearance"
  | "right_of_way_clearing"
  | "brush_clearing"
  | "scheduled_pruning_cycle"
  | "seasonal_inspection"
  | "mulch_refresh"
  | "health_monitoring"
  | "tree_care_plan"
  | "tree_report"
  | "inventory_report"
  | "risk_report"
  | "photo_documentation"
  | "before_after_documentation"
  | "gis_mapping_update"
  | "arborist_consultation"
  | "tree_valuation"
  | "preservation_planning"
  | "species_recommendation"
  | "legal_insurance_consulting";

export interface ServiceTypeDefinition {
  id: ServiceTypeId;
  label: string;
  category: ServiceCategory;
  description: string;

  allowedTargets: ServiceTargetType[];
  executionMode: ServiceExecutionMode;

  defaultBillingMode: ServiceBillingMode;
  defaultPriority: ServicePriorityDefault;

  requiresAssessmentFirst?: boolean;
  canBeRecurring?: boolean;
  canBeEmergency?: boolean;
  requiresFollowUp?: boolean;

  suggestedUnits?: Array<
    "hours" | "visits" | "trees" | "zones" | "stumps" | "loads" | "applications"
  >;

  defaultChecklist?: string[];
  defaultTags?: string[];

  writesHistoryAs:
    | "service"
    | "inspection"
    | "measurement"
    | "note"
    | "report";

  isActive: boolean;
}
```

---

## Example catalog entries

Proposed location: `lib/service-catalog.ts` (import path adjusted to your alias).

```ts
import { ServiceTypeDefinition } from "@/types/service-types";

export const SERVICE_CATALOG: Record<string, ServiceTypeDefinition> = {
  structural_pruning: {
    id: "structural_pruning",
    label: "Structural Pruning",
    category: "pruning_trimming",
    description: "Pruning to improve tree structure, form, and long-term stability.",
    allowedTargets: ["tree", "multi_tree", "zone"],
    executionMode: "one_time",
    defaultBillingMode: "per_tree",
    defaultPriority: "normal",
    canBeRecurring: true,
    requiresFollowUp: true,
    suggestedUnits: ["trees", "hours"],
    defaultChecklist: [
      "Review species and age",
      "Inspect defects and structure",
      "Define pruning objective",
      "Document cuts performed",
      "Schedule follow-up if needed",
    ],
    defaultTags: ["pruning", "structure", "training"],
    writesHistoryAs: "service",
    isActive: true,
  },

  annual_inspection: {
    id: "annual_inspection",
    label: "Annual Inspection",
    category: "inspection_assessment",
    description: "Routine scheduled inspection of tree condition and risk.",
    allowedTargets: ["tree", "multi_tree", "zone", "property", "site"],
    executionMode: "recurring",
    defaultBillingMode: "per_visit",
    defaultPriority: "normal",
    canBeRecurring: true,
    requiresFollowUp: false,
    suggestedUnits: ["visits", "trees", "hours"],
    defaultChecklist: [
      "Inspect canopy and trunk",
      "Review root zone and soil",
      "Assess health and risk",
      "Capture photos",
      "Record recommendations",
    ],
    defaultTags: ["inspection", "monitoring"],
    writesHistoryAs: "inspection",
    isActive: true,
  },

  tree_removal: {
    id: "tree_removal",
    label: "Tree Removal",
    category: "removal_stump",
    description: "Complete removal of a tree.",
    allowedTargets: ["tree", "multi_tree", "zone"],
    executionMode: "one_time",
    defaultBillingMode: "per_tree",
    defaultPriority: "high",
    canBeEmergency: true,
    requiresAssessmentFirst: true,
    suggestedUnits: ["trees", "hours"],
    defaultChecklist: [
      "Confirm scope and access",
      "Assess hazards",
      "Protect surrounding assets",
      "Remove tree",
      "Haul debris",
      "Update site condition",
    ],
    defaultTags: ["removal", "hazard", "crew"],
    writesHistoryAs: "service",
    isActive: true,
  },
};
```

---

## Optional Firestore catalog

If definitions must be admin-editable, mirror or source them from:

`service_catalog/{serviceTypeId}`

Otherwise a versioned in-repo catalog (above) is enough for v1.
