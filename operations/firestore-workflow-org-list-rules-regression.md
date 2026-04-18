# Known issue: Org workflow list drawers empty (Firestore rules regression)

**Status:** Mitigated in production by rules rollback (April 2026).  
**Severity:** High — owners and member+ users could not see Requests / Quotes / Jobs / Invoices in org list drawers while **Work** (`participant-work`) still showed the same records.

> **Deploy warning:** Until `app/firebase/firestore.rules` is reconciled with [`working-fire-rules.txt`](../../app/working-fire-rules.txt) and redeployed from git, a normal **`firebase deploy --only firestore:rules`** from the app branch could **re-break** production if the branch still contains the stricter rules. Coordinate deploys with this runbook.

---

## Symptoms

- User has a valid **team** `organizationId`, **`role: owner`** (or other member+ role), and an **active** subscription tier that includes the workflow pipeline (e.g. Small Team).
- Workflow documents in Firestore are correct (e.g. `quotes/{id}` has `organizationId` matching the team, `participantPrincipalIds` includes `uid:{actor}` and `org:{teamId}`).
- **`?drawer=participant-work`** (Work) **correctly** lists quotes, requests, etc.
- **`?drawer=quotes`** (and sibling org list drawers: requests, jobs, invoices) show **empty** states such as “No quotes yet…” — **not** necessarily the “role denied” or upgrade banners.
- Browser **Console** may show **no** obvious `permission-denied` text; failures can present as empty lists or failed queries depending on client behavior.

---

## Why two drawers behaved differently (not a data bug)

| Surface | How data is loaded | Rules on read |
|--------|-------------------|----------------|
| **Work** (`participant-work`) | Authenticated **`GET /api/workflow/participant-list`** → server uses **Firebase Admin SDK** to query `participantPrincipalIds` / assignee fields | Client Firestore rules are **not** applied to that server list path |
| **Org workflow drawers** (quotes, requests, jobs, invoices) | Client **`getDocs`** via `listQuotes` / `listRequests` / … in [`useQuotes`](../../app/hooks/useQuotes.ts), [`useRequests`](../../app/hooks/useRequests.ts), etc., gated by [`useWorkflowOrgListQueryEnabled`](../../app/hooks/useOrgWorkflowReadPermissions.ts) | Every document returned by the org-wide query must satisfy **deployed Firestore rules** |

So: **same Firestore data**, two different read paths. Work could “look fine” while org lists broke entirely when **client** list reads were denied or could not be proven safe for the query shape.

---

## Root cause (confirmed)

Deploying **stricter** workflow read rules (relative to the known-good snapshot) broke **org-wide** list reads for insider users. The regression set introduced additional conditions beyond **team membership** for non-participant workflow reads, notably:

- Requiring **`hasRole(['member','manager','admin','owner'])`** on **`users/{uid}.role`** together with **`workflowTeamAccess`**, and/or  
- Applying **`users.orgPolicy.blockOrgWorkflowReadEntities`** (v4 preset / deny overrides) via **`orgPolicyBlocksOrgWorkflowRead(entity)`** for entity-scoped org reads,

while list queries use patterns like `where('organizationId', '==', teamId)` plus `orderBy(...)`. Firestore security requires that **any document that could match the query** must pass read rules; tightening rules without aligning query constraints or simulator tests commonly results in **empty lists** or **failed queries** for collection scans.

**Mitigation that restored behavior:** rolling deployed Firestore rules back to the last **known-good** revision (saved as a snapshot in the app repo for reference).

---

## What we verified during investigation

1. **User document** — `role: owner`, `subscription.tier` / `status` appropriate for **`workflow.pipeline`** in the app; top-level **`organizationId`** aligned with team id when present.
2. **Sample quote document** — `organizationId` matched team id; `participantPrincipalIds` and `v3Workflow` looked correct.
3. **No reliable Console error** in all cases — still consistent with rules/query mismatch or silent client handling.
4. **Rollback of Firestore rules** — **resolved** org workflow list drawers in production.

---

## File references (links)

### Canonical rules in the app repository (source of truth for CI / deploy)

- **[`app/firebase/firestore.rules`](../../app/firebase/firestore.rules)** — Firestore rules as tracked in git (must stay in sync with what is **published** to Firebase after fixes).

### Known-good snapshot (post-rollback reference, April 2026)

- **[`app/working-fire-rules.txt`](../../app/working-fire-rules.txt)** — Saved copy of rules that **restored** org workflow list drawers. Use this to **diff** against `firebase/firestore.rules` before the next rules deploy.

> **Local path note:** In a workspace where `docs` and `app` are siblings (e.g. `Groundzy/docs` and `Groundzy/app`), the relative links above resolve from this file under `docs/operations/`. If your clone layout differs, open the same paths from your repo roots.

### Quick copy paths (typical Groundzy clone)

| Artifact | Path (relative to repo root) |
|----------|------------------------------|
| Tracked rules (app repo) | `app/firebase/firestore.rules` |
| Known-good snapshot (app repo) | `app/working-fire-rules.txt` |
| This runbook (docs repo) | `docs/operations/firestore-workflow-org-list-rules-regression.md` |

### Related application code (for engineers re-introducing stricter rules safely)

- [`app/hooks/useOrgWorkflowReadPermissions.ts`](../../app/hooks/useOrgWorkflowReadPermissions.ts) — `useWorkflowOrgListQueryEnabled`, org read matrix / presets (client gate; not the same as rules).
- [`app/hooks/useParticipantWorkflow.ts`](../../app/hooks/useParticipantWorkflow.ts) — Work list via API (bypasses client rules for listing).
- [`app/lib/server/participant-workflow-list.ts`](../../app/lib/server/participant-workflow-list.ts) — Server-side participant list implementation.
- [`app/lib/groundzy/policy/can-append-event.ts`](../../app/lib/groundzy/policy/can-append-event.ts) — Append pipeline `organizationId` scope check (`userData.organizationId ?? actorUid`).

### Product / policy context (docs repo)

- [`groundzy-v3-docs/05-data/roles-permissions-product-spec.md`](../groundzy-v3-docs/05-data/roles-permissions-product-spec.md) — Org list drawers vs participant Work (§4.2–4.3).
- [`groundzy-v3-docs/05-data/org-action-policy-matrix.md`](../groundzy-v3-docs/05-data/org-action-policy-matrix.md) — Org actions and presets (rules should match intent before deploy).

---

## Follow-up (engineering checklist)

1. **Diff** [`firebase/firestore.rules`](../../app/firebase/firestore.rules) against [`working-fire-rules.txt`](../../app/working-fire-rules.txt) and document every intentional change (especially workflow `allow read` branches for `requests` / `quotes` / `jobs` / `invoices`).
2. **Firestore Rules Playground** — simulate **list** reads for `quotes` with `organizationId == teamId` and `orderBy`, for roles: owner, member, **viewer**, and users with **`orgPolicy.blockOrgWorkflowReadEntities`** where applicable.
3. **Align** rules with [`org-action-policy-matrix.md`](../groundzy-v3-docs/05-data/org-action-policy-matrix.md) without breaking **collection query** provability.
4. **UI improvement (optional):** org list drawers treat “query disabled” and “query error” like empty data today; consider distinct empty / error / upgrade states (see [`app/app/drawers/quotes.tsx`](../../app/app/drawers/quotes.tsx) pattern).
5. After rules are fixed and tested, **re-deploy** from `firebase/firestore.rules` and remove or archive the temporary snapshot if no longer needed (or keep `working-fire-rules.txt` updated as a labeled baseline).

---

## Revision history

| Date | Action |
|------|--------|
| 2026-04 | Issue observed; investigation; production rules **rolled back** to contents of `working-fire-rules.txt`; this document added. |
