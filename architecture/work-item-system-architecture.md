# Work item system architecture

> **Superseded for system-of-record claims (Groundzy v3).** v3 is **event-first**: **Events** are the only canonical source of truth for actions; WorkItem-shaped data is a **projection**, not a parallel authority. See [`Groundzy v3/00-foundation/principles.md`](../../Groundzy%20v3/00-foundation/principles.md) and [`Groundzy v3/05-data/event-system.md`](../../Groundzy%20v3/05-data/event-system.md). This file remains useful for **legacy codebase** behavior and **ServiceType** references.

**ServiceType** is taxonomy (see [ServiceType model spec](../reference/service-type-model-spec.md)). **Below:** historical framing where **WorkItem** was described as **system of record** for planning, scheduling, quoting, jobs, invoicing, zone work, and tree history output—**not** the v3 model.

That matches Groundzy’s existing split:

- **Tree history** — records (service, inspection, measurement, note): good **output log**, not the main execution object
- **Zone services** — bulk / zone-scoped work (candidate to unify under WorkItem)
- **CRM** — request → quote → job → invoice

---

## Recommended architecture

### Rule of thumb

| Concept | Role |
|--------|------|
| **ServiceType** | What kind of work this is |
| **WorkItem** | One actual piece of work to be done |
| **Workflow docs** | Commercial / project **containers** around work items |

Workflow containers:

- request  
- quote  
- job  
- invoice  

### Relationships

- A **quote** has many work items  
- A **job** has many work items  
- A **zone** can have one work item affecting many trees  
- A **tree** can have many work items over time  
- **Completed** work items write a **summarized** record into tree history  

Keeping taxonomy in **ServiceType** avoids overloading tree history with scheduling and commercial state. History stays chronological and readable.

---

## Part 1: ServiceType (summary)

The full enum and `ServiceTypeDefinition` interface live in [service-type-model-spec.md](../reference/service-type-model-spec.md). Treat that file as the canonical **catalog** shape.

---

## Part 2: Unified WorkItem model

### Core design principle

A `WorkItem` is **one actionable unit** of field or consulting work, whether it originates from:

- a tree, zone, property, or site  
- a request, quote, or job  

One system for planning, ops, and CRM.

### Type definitions

Proposed location: `types/work-item.ts`.

```ts
import { ServiceTypeId } from "@/types/service-types";

export const WORK_ITEM_STATUSES = [
  "draft",
  "planned",
  "quoted",
  "approved",
  "scheduled",
  "in_progress",
  "blocked",
  "completed",
  "cancelled",
  "archived",
] as const;

export type WorkItemStatus = typeof WORK_ITEM_STATUSES[number];

export const WORK_ITEM_PRIORITIES = [
  "low",
  "normal",
  "high",
  "urgent",
  "emergency",
] as const;

export type WorkItemPriority = typeof WORK_ITEM_PRIORITIES[number];

export const WORK_ITEM_TARGET_KINDS = [
  "tree",
  "zone",
  "property",
  "client",
  "site",
  "mixed",
] as const;

export type WorkItemTargetKind = typeof WORK_ITEM_TARGET_KINDS[number];

export const WORK_ITEM_SOURCE_TYPES = [
  "manual",
  "tree_history_followup",
  "inspection_recommendation",
  "weather_recommendation",
  "request",
  "quote",
  "job",
  "recurring_plan",
  "zone_template",
  "bulk_action",
] as const;

export type WorkItemSourceType = typeof WORK_ITEM_SOURCE_TYPES[number];

export const WORK_ITEM_OUTCOME_TYPES = [
  "service_completed",
  "inspection_completed",
  "monitoring_completed",
  "report_delivered",
  "no_action_needed",
  "deferred",
  "cancelled",
] as const;

export type WorkItemOutcomeType = typeof WORK_ITEM_OUTCOME_TYPES[number];

export interface WorkItemTargetRef {
  kind: "tree" | "zone" | "property" | "client" | "site";
  id: string;
  label?: string;
}

export interface WorkItemCommercialRefs {
  requestId?: string;
  quoteId?: string;
  jobId?: string;
  invoiceId?: string;
}

export interface WorkItemSchedule {
  dueAt?: string;       // ISO
  startAt?: string;     // ISO
  endAt?: string;       // ISO
  allDay?: boolean;
  recurrenceRuleId?: string;
}

export interface WorkItemAssignment {
  assignedUserIds?: string[];
  assignedTeamIds?: string[];
  crewLabel?: string;
}

export interface WorkItemEstimate {
  estimatedHours?: number;
  estimatedUnits?: number;
  unitLabel?: string; // trees, visits, applications, etc.
  estimatedCost?: number;
  estimatedPrice?: number;
  currency?: string;
}

export interface WorkItemCompletion {
  completedAt?: string;
  completedByUserId?: string;
  outcomeType?: WorkItemOutcomeType;
  outcomeNotes?: string;
  followUpRecommended?: boolean;
  followUpWorkItemId?: string;
}

export interface WorkItemSnapshot {
  treeHealthBefore?: string;
  treeRiskBefore?: string;
  treeHealthAfter?: string;
  treeRiskAfter?: string;
  weatherSummary?: string;
}

export interface WorkItemSource {
  type: WorkItemSourceType;
  sourceId?: string;
  sourceLabel?: string;
}

export interface WorkItem {
  id: string;
  organizationId: string;

  serviceTypeId: ServiceTypeId;
  title: string;
  description?: string;

  status: WorkItemStatus;
  priority: WorkItemPriority;

  targetKind: WorkItemTargetKind;
  targets: WorkItemTargetRef[];

  source: WorkItemSource;
  commercialRefs?: WorkItemCommercialRefs;

  schedule?: WorkItemSchedule;
  assignment?: WorkItemAssignment;
  estimate?: WorkItemEstimate;
  completion?: WorkItemCompletion;
  snapshot?: WorkItemSnapshot;

  checklist?: Array<{
    id: string;
    label: string;
    isDone: boolean;
  }>;

  attachments?: Array<{
    id: string;
    url: string;
    type: "image" | "pdf" | "document" | "other";
    label?: string;
  }>;

  tags?: string[];

  createdAt: string;
  createdByUserId: string;
  updatedAt: string;
  updatedByUserId: string;

  archivedAt?: string;
  archivedByUserId?: string;
}
```

---

## Why this fits Groundzy

### Trees

Examples: structural pruning, annual inspection, fertilization — each as a work item on a given tree.

### Zones

Examples: mulch refresh for a zone, inventory tagging, storm inspection for a grove. Aligns with `zone_services` but backed by a single work model.

### CRM

- Request holds **proposed** work items  
- Quote **prices** the same work items  
- Job **schedules and assigns** them  
- Invoice **bills** completed / approved work items  

Avoid duplicating service logic inside each document type.

### Tree history

On completion, write a **compact** history line per affected tree (service / inspection / note / report per `ServiceTypeDefinition.writesHistoryAs`). WorkItem remains operational; history stays an audit trail.

---

## Relationship model (best practice)

Do **not** embed full service logic in requests, quotes, jobs, or invoices.

| Collection | Role |
|-----------|------|
| `requests/{id}` | Customer intake / origin |
| `quotes/{id}` | Pricing / approval wrapper |
| `jobs/{id}` | Scheduling / execution wrapper |
| `invoices/{id}` | Billing wrapper |
| `work_items/{id}` | **Canonical** units of work |

---

## Minimal linking strategy

```ts
interface Request {
  id: string;
  workItemIds: string[];
}

interface Quote {
  id: string;
  workItemIds: string[];
}

interface Job {
  id: string;
  workItemIds: string[];
}

interface Invoice {
  id: string;
  workItemIds: string[];
}
```

You can still denormalize summaries on those documents for list views; **WorkItem** stays canonical.

---

## Status flow

Happy path:

`draft` → `planned` → `quoted` → `approved` → `scheduled` → `in_progress` → `completed`

Side exits:

`blocked` · `cancelled` · `archived`

### Meaning (concise)

| Status | Meaning |
|--------|---------|
| **draft** | Internal idea, not yet proposed |
| **planned** | Defined as work, not yet in a quote |
| **quoted** | Included in a quote |
| **approved** | Customer approved |
| **scheduled** | Date / crew assigned |
| **in_progress** | Active in the field |
| **completed** | Done; outcome recorded |

This separates commercial approval from field execution instead of one overloaded status.

---

## Targeting: single vs bulk

Use `targets[]` so one work item can represent:

- one tree  
- many trees  
- a zone  
- a property  

### Examples

Zone mulch refresh:

```ts
{
  targetKind: "zone",
  targets: [{ kind: "zone", id: "zone_123", label: "North Bed" }]
}
```

Bulk pruning cycle:

```ts
{
  targetKind: "mixed",
  targets: [
    { kind: "tree", id: "tree_1" },
    { kind: "tree", id: "tree_2" },
    { kind: "tree", id: "tree_3" }
  ]
}
```

---

## Completion writeback to trees

When `WorkItem.status === "completed"`:

### For each tree target

Append a history record shaped like:

```ts
interface TreeServiceHistoryRecord {
  id: string;
  workItemId: string;
  serviceTypeId: ServiceTypeId;
  performedAt: string;
  performedByUserId?: string;
  summary: string;
  outcomeNotes?: string;
}
```

(Align field names with existing tree history types in the app.)

### Zone- or property-only targets

- Optional zone activity log  
- Optional child per-tree records if you need inventory-level detail  

---

## Recurring care plans (separate from WorkItem)

Plans **generate** work items; they are not work items themselves.

Proposed location: `types/recurring-plan.ts`.

```ts
import { ServiceTypeId } from "@/types/service-types";

export interface RecurringPlan {
  id: string;
  organizationId: string;

  title: string;
  serviceTypeId: ServiceTypeId;

  targetKind: "tree" | "zone" | "property" | "site";
  targetIds: string[];

  frequency: {
    unit: "week" | "month" | "quarter" | "year";
    interval: number;
  };

  defaultPriority: "low" | "normal" | "high" | "urgent";
  defaultAssigneeUserIds?: string[];

  nextRunAt: string;
  isActive: boolean;

  createdAt: string;
  updatedAt: string;
}
```

---

## What should replace `zone_services`?

### Option A (preferred)

Drop a separate `zone_services` concept; use **`work_items`** with `targetKind = "zone"` (and `targets` pointing at the zone).

### Option B (transitional)

Keep `zone_services` as thin legacy wrappers that point at or mirror `work_items`.

Option A is simpler long term.

---

## Suggested Firestore shape

```txt
work_items/{workItemId}
recurring_plans/{planId}
service_catalog/{serviceTypeId}   // optional if admin-editable
```

Keep existing collections as today, for example:

```txt
requests/{id}
quotes/{id}
jobs/{id}
invoices/{id}
trees/{id}
zones/{id}
properties/{id}
clients/{id}
```

---

## Migration path (phased)

| Phase | Action |
|-------|--------|
| **1** | Introduce ServiceType catalog + WorkItem **types** only (no behavior change required everywhere at once). |
| **2** | When a user adds a “service” or “inspection” from a tree, create a **work item** first. |
| **3** | On completion, mirror a **summary** into tree history. |
| **4** | Point quote / job forms at **`workItemIds`**. |
| **5** | Replace zone service handling with **WorkItem**. |

---

## Canonical model (summary)

- **ServiceType** — controlled taxonomy  
- **WorkItem** — canonical unit of work  
- **RecurringPlan** — generator for future work  
- **Tree history** — audit / output log  
- **Request / quote / job / invoice** — workflow and commercial containers  

### Spine sentence

> **Everything actionable is a WorkItem. Everything historical is a Record. Everything billable or schedulable references WorkItems.**

---

## Next implementation step

Promote the snippets in this doc and in [service-type-model-spec.md](../reference/service-type-model-spec.md) to production **`types/service-types.ts`**, **`types/work-item.ts`**, **`lib/service-catalog.ts`**, helpers, and migration notes in code or Firestore rules as you wire Phase 1–5.
