# Sales Portal — Client account detail page UX audit

**Scope:** Human usability, information architecture, visual hierarchy, and layout of the rep-facing client account view (single-account “CRM record” screen).

**Source reviewed:** `sales-portal/app/(rep)/clients/[accountId]/page.tsx` (`ClientAccountDetailPage`), plus `app/(rep)/layout.tsx` (shell only).

**Route:** `/clients/[accountId]` (legacy redirect from `/crm/:accountId` per `next.config.ts`).

**Date:** 2026-05-17

---

## Executive summary

The client detail page is a **single, very long scroll** (~1,400+ lines of JSX in one component) that mixes **high-signal narrative data** (status, pipeline, timeline) with **heavy inline editing** (many pill groups, multiple save buttons, address and state forms). There is **no clear visual “story”** from top to bottom—sections appear in an order that favors implementation bundles rather than a rep’s mental model (understand account → decide next action → log work → read history).

**Readability suffers** from uneven card usage (some blocks are bare paragraphs), **tiny typography** in many labels and metadata (`text-[10px]`, `text-[11px]`), **inconsistent date formatting** (raw ISO substrings vs locale), and **no in-page navigation** (no sticky header, tabs, or table of contents), so **finding a specific fact requires scrolling and visual search**.

**Layout exists minimally** (`max-w-4xl`, occasional `grid`), but **there is almost no above-the-fold summary** and **no persistent structure** that anchors “where am I on this page.” The result matches the complaint: functional, but **hard for a human to scan quickly**.

---

## Rep jobs-to-be-done (implicit goals)

Typical on this screen:

1. **Orient:** Who is this, what stage are we in, what’s urgent?
2. **Act:** Log a call, move a deal, complete a workflow task, update health, post a team note.
3. **Research:** Intelligence, library Q&A, Google/Maps context, contact details.
4. **History:** Timeline of interactions and comments.

The current ordering only partially supports (1) before dropping the user into long editable forms.

---

## Section inventory (top → bottom)

Approximate order as implemented:

| # | Section | Container | Notes |
|---|----------|-----------|--------|
| 1 | Back link, title, status, optional `updatedAt`, Archive | Header row | Sparse summary |
| 2 | Account phones / emails | 2-col grid, **cards only if data exists** | If both empty, this area is blank |
| 3 | Website | Plain `<p>` | Not grouped with contact |
| 4 | Google Place snapshot / rating | Plain `<p>` | Not grouped with Location |
| 5 | **Labels & notes** | Large card: tags, follow-up due, **Save labels**, then notes form, then primary contact form | Three workflows in one card |
| 6 | **Location** | Card: manual address + state dropdown | Separate from website/maps block |
| 7 | **Workflow tasks** | Card, **full width** in a 2-col grid wrapper | Col-span 2 breaks grid rhythm |
| 8 | **Account intelligence** | Card, col-span 2 | Dense; raw field keys |
| 9 | **Onboarding health** | Card, one column in grid | Score + signals |
| 10 | **Team comments** | Card | Short list + composer |
| 11 | **Library copilot** | Card | RAG ask |
| 12 | **Opportunities** | Full-width card | Add form + list |
| 13 | **Log activity** | Full-width card | Another large form |
| 14 | **Timeline** | **Not a card** | Visually disconnected from “Log activity” |

This sequence interleaves **editing**, **collaboration**, **AI**, and **pipeline** without strong group labels (e.g. “Work,” “Record,” “Insights”).

---

## Critical issues

### 1. No information hierarchy for “glanceable” account state

The header shows name and lifecycle-oriented status, but **does not surface**:

- Next action due (even though the page edits `nextActionAt` and next-step tags deeper in “Labels & notes”).
- Open opportunity stages or counts as a compact summary.
- Workflow tasks count or the nearest due task.

Reps must **scroll into the middle of the page** to see urgency. The most actionable fields are **below the fold** inside a card titled “Labels & notes,” which buries “follow-up / due” under many tag pickers.

### 2. “One column of everything” with uneven treatment of blocks

Primary container: `mx-auto max-w-4xl space-y-8`. That is a reasonable reading width, but:

- **Phones/emails** use cards; **website** and **Google rating** are loose text—not scannable as a single “Contact & presence” cluster.
- **Timeline** is rendered **without** `gz-card`, unlike almost everything else, so it feels like an appendix.
- **Workflow tasks** and **intelligence** use `lg:col-span-2`, which visually separates them from **health / comments / copilot** in the same grid—creating an **uneven two-column band** that is easy to misread on large screens.

### 3. Cognitive overload: many independent save surfaces

Within “Labels & notes,” users face:

- Lifecycle/outreach pills + five more multiselect groups + datetime for next step → **Save labels**
- Notes textarea → **Save notes**
- Primary contact fields → **Save primary contact**

That is **three submit actions** in one card, plus separate saves for address and state in Location. **Error state** (`err`) is **global** for many operations—a failure saving labels can appear at the top while the user is focused on notes or location, causing **poor error locality**.

### 4. Typography and metadata hurt quick reading

Patterns that reduce legibility:

- Widespread **`text-[10px]`** and **`text-[11px]`** for section hints and timeline stamps (below typical accessible body text recommendations).
- **`updatedAt`** shown as a raw string from the API without user-friendly formatting.
- Timeline uses **`absolute` left positioning** with a fixed width for timestamps (`-left-24 w-24`). On narrow viewports this is fragile and can **overlap or clip** relative to the border line.
- **Intelligence “Business profile”** renders `Object.entries` with **raw camelCase keys** as labels (`serviceArea:` style). Humans expect human labels, not schema names.

### 5. Weak identity and trust cues in collaboration

- Comments show **`authorUid` truncated** instead of a display name—hard to trust who said what without cross-reference.
- Interaction list has similar issues: good content structure, but **no visual grouping by day** or relative time (“Yesterday”).

### 6. Findability: no anchors, no search-within-page

There is **no** sticky sub-nav, **no** tabs (Overview / Pipeline / Activity / Settings), **no** “jump to” links. For a page this long, **scrolling is the only navigation**.

### 7. Implementation smell (maintainability affects UX iteration)

The entire view is **one default export** with many `useState` hooks and inline handlers. That makes it hard to introduce layout regions, consistent empty states, or progressive disclosure without risk. **Splitting by domain** (header summary, editable metadata, pipeline, activity) would unlock layout improvements.

---

## Secondary / moderate issues

- **Empty phones and emails:** The two-column block disappears entirely—no “No phone on file” placeholder, unlike opportunities’ empty state messaging.
- **Library copilot** embedded mid-page: useful, but **competes** with account-specific intelligence and **distracts** from operational workflows. Consider placement in a drawer or lower priority region.
- **Opportunities list:** Trial editor is **nested per row** with extra borders—visually busy when multiple opps exist.
- **Global `busy` flag:** Many actions set `busy`, which can **disable unrelated controls** across the page and make the UI feel “frozen” during one save.

---

## What works reasonably well

- **Clear card primitive** (`gz-card`) for most sections—consistent padding and borders when used.
- **Location duplicate handling** surfaces `locDuplicateId` with a link to the conflicting account—good actionable error pattern.
- **Opportunities** empty state copy is explicit.
- **Activity logging** form fields are labeled; interaction types are understandable.

---

## Recommendations (prioritized)

### P0 — Quick wins (structure without full redesign)

1. **Add a compact “Account overview” strip** under the title: next step due, # open opps / stages, # open workflow tasks, primary phone/email links. Pull from data already on the page or from existing API responses.
2. **Introduce sticky section tabs or a mini table of contents** (Overview · Pipeline · Activity · Insights · Admin) with anchor IDs—reduces scroll hunting immediately.
3. **Normalize typography:** replace `text-[10px]` / borderline sizes with at least `text-xs` (12px) for secondary labels; keep timestamps **human-readable** (`toLocaleString` or date-fns).
4. **Unify “Contact & web & maps”** into one card: phones, emails, website, Google snippet, link to Maps.
5. **Wrap Timeline in the same card system** as other sections for visual consistency.

### P1 — Reduce cognitive load

6. **Split “Labels & notes”** into separate cards: **Tags & next step**, **Notes**, **Primary contact**—each with a local success/error affordance.
7. **Scope loading states:** prefer per-section `metaBusy`, `locBusy`, etc., instead of one `busy` for all mutations where feasible.
8. **Map intelligence keys** to readable labels and order fields by importance (size, market, incumbent, etc.).

### P2 — Deeper layout

9. **Two-column “workbench” layout on desktop:** left column “record + pipeline,” right column “tasks + activity + insights,” with Timeline full width below—or use **tabs** for “Record” vs “Activity.”
10. **Extract subcomponents** (`AccountHeader`, `ContactStrip`, `LabelsCard`, …) to make layout changes safe and testable.

---

## Conclusion

The page is **feature-complete but human-hostile for scanning**: long single scroll, mixed concerns in one card, microscopic helper text, and missing **above-the-fold operational summary**. Addressing **wayfinding** (anchors/tabs/sticky summary) and **grouping** (contact block, split save surfaces) would yield the largest usability gain before visual polish.

---

## Appendix: code pointers

| Topic | Location |
|--------|----------|
| Root layout wrapper | `753:754:sales-portal/app/(rep)/clients/[accountId]/page.tsx` (`max-w-4xl space-y-8`) |
| Header + archive | `753:777:sales-portal/app/(rep)/clients/[accountId]/page.tsx` |
| Labels & notes (combined) | `867:1018:sales-portal/app/(rep)/clients/[accountId]/page.tsx` |
| Grid with workflow + intel + health + comments + copilot | `1076:1245:sales-portal/app/(rep)/clients/[accountId]/page.tsx` |
| Opportunities | `1248:1331:sales-portal/app/(rep)/clients/[accountId]/page.tsx` |
| Log activity + Timeline | `1333:1419:sales-portal/app/(rep)/clients/[accountId]/page.tsx` |
| Intelligence raw keys | `1126:1132:sales-portal/app/(rep)/clients/[accountId]/page.tsx` |
| Comment author display | `1191:1193:sales-portal/app/(rep)/clients/[accountId]/page.tsx` |
