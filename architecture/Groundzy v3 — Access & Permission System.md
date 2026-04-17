# Groundzy v3 — Access & Permission System

**Status:** Canonical (v3)
**Purpose:** Define a single, consistent access model across UI, API, Firestore rules, and event system.
**Scope:** All product surfaces (map, CRM, workflow, AI, sharing)

---

# 1. Core Principle

> **Tier is not access.**

* **Tier** = product packaging (what tools are available globally)
* **Access** = what a user can do on a specific resource

All permissions must be derived from:

```
Actor + Resource + Action → Policy → Result
```

---

# 2. Access Layers (Execution Order)

All access decisions follow this strict order:

1. **Authentication** — is the user signed in?
2. **Resource relationship** — how is the user related to this item?
3. **Policy** — what actions are allowed for that relationship?
4. **Entitlements (tier)** — is the feature globally available?
5. **UI rendering** — display based on final decision

⚠️ UI must NEVER introduce new access logic.

---

# 3. Actor Model

```ts
type AccessActor = {
  uid: string | null;

  effectiveTier:
    | "Home"
    | "Plus"
    | "Pro"
    | "Small Team"
    | "Mid Team"
    | "Large Team"
    | "Enterprise";

  entitlements: {
    workflowPipeline: boolean;
    crmProperties: boolean;
    crmClients: boolean;
    workItemsReadOrg: boolean;
    mapLayersJobs: boolean;
  };

  orgMemberships: Array<{
    organizationId: string;
    /** Aligns with `TeamRole` in product types — includes read-only org members (`viewer`). */
    role: "owner" | "admin" | "manager" | "member" | "viewer";
  }>;

  principalKeys: string[]; // uid:, org:, email:
};
```

---

# 4. Resource Roles

Derived from each resource:

```ts
type ResourceRole =
  | "owner"
  | "org_admin"
  | "org_member"
  | "assignee"
  | "participant"
  | "shared_reader"
  | "none";
```

### Sources of truth:

* `organizationId`
* `participantPrincipalIds`
* assignee fields
* creator
* sharing permissions

---

# 5. Action Model

All permissions are evaluated per **explicit action**:

```ts
type Action =
  | "workflow.request.create"
  | "workflow.request.read"
  | "workflow.request.update"
  | "workflow.request.comment"
  | "workflow.request.upload"
  | "workflow.quote.approve"
  | "workflow.invoice.pay"
  | "workflow.job.schedule"
```

No generic “access” flags allowed.

---

# 6. Permission Engine

```ts
function can(
  actor: AccessActor,
  action: Action,
  resource?: any
): boolean
```

This is the **only source of truth** for permissions.

---

# 7. Global vs Record Permissions

## Global (entitlement-based)

Controls access to product surfaces:

| Capability            | Controlled by                 |
| --------------------- | ----------------------------- |
| Create workflow items | `workflowPipeline` **or** Pro **solo** scope (see below) |
| CRM                   | `crmClients`, `crmProperties` |
| Map job layers        | `mapLayersJobs`               |

**Pro solo workflow create:** Users on the **Pro** tier may create workflow entities (requests → quotes → jobs → invoices) scoped to **personal org**—i.e. `organizationId` (and related fields) match **solo** patterns the app already uses (e.g. `organizationId == actor uid`, solo `databaseCode`). **Team** tiers continue to use org workflow via `workflowPipeline` and team membership. Policy and append checks must allow this path, not only `workflowPipeline`.

---

## Record-level (relationship-based)

Controls access to specific records:

| Action    | Controlled by |
| --------- | ------------- |
| View item | relationship  |
| Comment   | relationship  |
| Upload    | relationship  |
| Approve   | relationship  |
| Edit      | role + policy |

---

# 8. Workflow Policy Matrix

## Request

| Role             | Read | Comment | Upload | Edit    | Status  |
| ---------------- | ---- | ------- | ------ | ------- | ------- |
| Org admin/member | Yes  | Yes     | Yes    | Yes     | Yes     |
| Assignee         | Yes  | Yes     | Yes    | Limited | Limited |
| Participant      | Yes  | Yes     | Yes    | No      | No      |
| Shared reader    | Yes  | No      | No     | No      | No      |

---

## Quote

| Role             | Read | Comment | Upload | Edit    | Approve |
| ---------------- | ---- | ------- | ------ | ------- | ------- |
| Org admin/member | Yes  | Yes     | Yes    | Yes     | Yes     |
| Assignee         | Yes  | Yes     | Yes    | Limited | No      |
| Participant      | Yes  | Yes     | Yes    | No      | Yes     |
| Shared reader    | Yes  | No      | No     | No      | No      |

---

## Job

| Role             | Read | Comment | Upload | Edit    | Schedule |
| ---------------- | ---- | ------- | ------ | ------- | -------- |
| Org admin/member | Yes  | Yes     | Yes    | Yes     | Yes      |
| Assignee         | Yes  | Yes     | Yes    | Limited | Limited  |
| Participant      | Yes  | Yes     | Yes    | No      | No       |
| Shared reader    | Yes  | No      | No     | No      | No       |

---

## Invoice

| Role             | Read | Comment | Download | Edit | Pay |
| ---------------- | ---- | ------- | -------- | ---- | --- |
| Org admin/member | Yes  | Yes     | Yes      | Yes  | N/A |
| Assignee         | Yes  | Yes     | Yes      | No   | N/A |
| Participant      | Yes  | Yes     | Yes      | No   | Yes |
| Shared reader    | Yes  | No      | Yes      | No   | No  |

---

# 9. UI Rules

## Drawer Types

### Global drawers (tier-based)

* requests
* quotes
* jobs
* invoices
* CRM

→ gated by **entitlements**

---

### Record drawers (data-based)

* view-request
* view-quote
* view-job
* view-invoice

→ gated by **can(action, resource)**

---

## UI Example

Use **entity-typed** access (e.g. `RequestAccess`, `QuoteAccess`) from the server. When `canRead` is false, branch on **`reason`** (see §10), not a single generic empty state.

```ts
const { access } = useWorkflowResourceAccess("request", requestId);

if (!access?.canRead) {
  if (access?.reason === "no_relationship") return <UpgradeOrEmpty />;
  if (access?.reason === "forbidden") return <SupportError />;
  if (access?.reason === "missing_entitlement") return <PlanUpgrade />;
  return null;
}

return (
  <>
    {access.canUpdate && <EditButton />}
    {access.canComment && <CommentBox />}
    {access.canUpload && <Upload />}
  </>
);
```

---

# 10. Server Contract

**Access is computed on the server** (Firebase Admin SDK load + policy). The client must not treat Firestore “I could read this doc” or client-only `can()` as the final gate for record views; client `can()` is **optimistic only** when shared modules exist.

All workflow resource GETs return a **discriminated** shape with **entity-typed** `access`:

```ts
type ReadDenialReason =
  | "no_relationship"
  | "forbidden"
  | "missing_entitlement";

type WorkflowReadGate = {
  canRead: boolean;
  reason?: ReadDenialReason; // set when canRead is false (production)
};

// Example: request (fields match policy matrix; evolve per entity)
type RequestAccess = WorkflowReadGate & {
  canUpdate: boolean;
  canComment: boolean;
  canUpload: boolean;
  canChangeStatus: boolean;
};

// Response
{
  entity: "request",
  data: Request,
  access: RequestAccess
}
```

**Read denial reasons**

| `reason`               | Typical UX                                      |
| ---------------------- | ----------------------------------------------- |
| `no_relationship`      | Upgrade / marketing / empty state (no tie to row) |
| `forbidden`            | Error / support (unexpected deny)               |
| `missing_entitlement`  | Plan upgrade (global entitlement blocks surface) |

Server responses are **authoritative**.

---

## Participant identity (principal keys)

* **Primary match for signed-in users:** `uid:<firebaseUid>` on `participantPrincipalIds`.
* **`email:` keys** are not used in production writers today; if ever added, document claim/migration on signup and avoid ghost access (see implementation plan).

---

# 11. Firestore Rules (v3 Direction)

Rules enforce:

* read access (participants allowed)
* org-based writes
* scoped mutation paths

Participants:

* ✅ read
* ❌ full document update
* ✅ allowed for scoped actions (comments, uploads, approvals via controlled paths)

---

# 12. Event System Integration

Event permissions must be **action-based**, not tier-based.

Replace:

```
workflowPipeline → blanket allow/deny
```

With:

```
can(actor, "workflow.quote.approve", resource)
```

Examples:

| Action         | Allowed     |
| -------------- | ----------- |
| create request | Teams       |
| approve quote  | Participant |
| edit pricing   | Org admin   |
| pay invoice    | Participant |

---

# 13. Replacement of Legacy Patterns

## REMOVE

* `useTeamsOnlyAccess()`
* tier-only gating for view drawers
* direct `"Pro"` / `"Teams"` checks

---

## REPLACE WITH

* `useGlobalEntitlements()`
* `useResourceAccess(resource)`
* `can(actor, action, resource)`

---

# 14. Definitions

| Term          | Meaning                   |
| ------------- | ------------------------- |
| Entitlement   | Global product access     |
| Resource role | Relationship to record    |
| Policy        | Rules per action          |
| Access        | Final computed permission |

---

# 15. Non-Negotiable Rules

1. Tier NEVER directly gates record views
2. Participant is a first-class role
3. Actions are explicit (no generic access flags)
4. Server returns **entity-typed** access with every resource load (Admin SDK + policy); client optimistic `can()` only
5. UI does not invent permissions
6. Event permissions are action-based
7. When `canRead` is false, responses include a **`reason`** so the UI does not conflate “no relationship” with “server error”

---

# 16. System Summary

> **Global tools are unlocked by entitlement.
> Specific records are unlocked by relationship.
> Specific actions are unlocked by policy.**

This is the Groundzy v3 access model.
