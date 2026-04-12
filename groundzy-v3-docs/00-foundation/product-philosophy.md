# Product Philosophy

How Groundzy v3 should be **built** and **felt**.

## Map-first, context-rich

The map is not a decorative backdrop. It is the **primary frame** for where trees, zones, and properties live. Actions should reinforce spatial and relational context—users should rarely wonder “where does this belong?”

## One mental model, many modes

Homeowner, professional, and team modes differ in **depth and permission**, not in **parallel universes**. The product should feel like **one platform** with clear depth, not unrelated mini-apps stitched together.

## System before screen

We design **entities, relationships, and rules** first—then surfaces. Features that look good in a mock but require a second, shadow data model are rejected early. The legacy app’s pain around duplicated history paths and mirrored workflow rows is exactly what v3 avoids by construction (see `principles.md`).

## Consistency is a feature

Predictable layout, navigation, loading and empty states, and form behavior reduce cognitive load and support trust—especially for field use and for teams onboarding new staff. A shared design system is not optional decoration; it is **product quality**.

## Progressive depth

Expose complexity **when the user’s tier and task require it**. Defaults stay simple; power sits one deliberate step away—not hidden behind inconsistent patterns.

## Evidence and restraint

We inherit **validated workflows and business rules** from the legacy codebase as **requirements and lore**, not as excuses to copy every implementation detail. We prefer conservative, testable claims about behavior—consistent with engineering norms in `Groundzy v3/rules.md`.

## Accessible, international-ready

Language and locale matter for a product used in the field and across regions. Centralized i18n and thoughtful copy are part of the experience, not a bolt-on.

## Security and trust by design

Subscriptions, org boundaries, sharing, and data visibility are **first-class**—not retrofitted after the UI ships.
