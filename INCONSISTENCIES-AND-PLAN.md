# Pricing Inconsistencies & Fix Plan

Document: Audit of pricing inconsistencies across **auth.groundzy**, **groundzy.com**, and **app.groundzy**. Includes code quality fixes and tier messaging alignment.

**Date: 2026-04-10** (last reviewed with codebase)  
**Status: DECISIONS LOCKED — rollout implemented (see Execution checklist below)**  
**Note:** Part I code-quality fixes exist on a **local auth.groundzy branch**; merge/deploy separately.

**Issue IDs in this doc:** Part I uses items 1–5 (code quality). Part II uses **#1–#7** (messaging + feature gaps). The **Summary Table (Issues → Fixes)** uses a single sequence **1–10** (code items 1–5, then messaging/gaps 6–10). When cross-referencing, use the Summary Table row number for “issue 6 = Plus positioning,” not Part II’s “Issue #6” (API capability).

---

## Summary

1. **Code Quality** (5 issues) — fixed in **auth.groundzy** (local branch; not necessarily merged).
2. **Tier Messaging** — **Done:** groundzy.com FAQ (team tiers, Enterprise, Pro “workflow”), species/catalog copy (78,000+ taxa / 1,000+ catalog), “solo workflow” removed from marketing; **auth** Teams description + **Enterprise** quote-only column; **docs** updated.
3. **Feature Enforcement (API / white-label / SLA)** — **No new `capabilities.ts` gates**; marketing promises removed instead.

## Execution checklist (completed)

- [x] `groundzy.com/index.html` — JSON-LD + visible FAQ: team tiers stripped of API/premium/wl/SLA; Enterprise = quote + white-label / dedicated CS; Pro = “workflow”; Identifying Wand = 78,000+ taxa.
- [x] Other **groundzy.com** pages: `for-homeowners.html`, `for-landscapers.html`, `for-schools.html`, `for-arborists.html`, blog posts — 1,200+ species claims updated (Wand vs Catalog).
- [x] **auth.groundzy** — `PlanTier` includes `enterprise`; pricing table fifth column; onboarding: Enterprise → address → units → `mailto:` + redirect to app; Stripe unchanged (Enterprise never hits checkout).
- [x] **app.groundzy** — [docs/PRICING-REFERENCE.md](PRICING-REFERENCE.md) + this file aligned to decisions.

---

## PART I: CODE QUALITY ISSUES (FIXED ✅)

All five issues have been remedied in auth.groundzy:

### 1. ✅ Badge JSX Duplication → Extracted Component

**What was wrong:**
- Both "Most Popular" and "Recommended for you" badges used nearly identical JSX (16 lines of duplication)

**What was fixed:**
- Created `PricingBadge` component in `components/custom-pricing-table.tsx` (lines 31–53)
- Both badges now render via: `<PricingBadge icon={...} label={...} color={...} />`
- Result: -8 lines of code, +1 reusable component

**Files changed:**
- `components/custom-pricing-table.tsx` (lines 31–188)

---

### 2. ✅ Redundant `?? 0` Fallback → Removed

**What was wrong:**
```ts
for (const [tier, score] of Object.entries(ROLE_SCORES[role])) {
  scores[tier as PlanTier] += score ?? 0;  // ← unnecessary
}
```
`Object.entries()` on a `Partial<TierScores>` only yields defined values; the `?? 0` was dead code.

**What was fixed:**
```ts
scores[tier as PlanTier] += score;  // ← clean
```

**Files changed:**
- `lib/onboarding-tier-suggestion.ts` (lines 44, 49, 54)

---

### 3. ✅ Awkward Badge Condition → Simplified

**What was wrong:**
```tsx
{suggestedTier === tier && !("mostPopular" in plan && plan.mostPopular) && (
  // JSX...
)}
```
Double-negation hard to parse.

**What was fixed:**
```tsx
{"mostPopular" in plan && plan.mostPopular ? (
  <PricingBadge icon={Check} label={t("pricing.mostPopular")} color="tea-green" />
) : suggestedTier === tier ? (
  <PricingBadge icon={Sparkles} label={t("pricing.recommendedForYou")} color="green-yellow" />
) : null}
```
Ternary makes mutual exclusivity clear.

**Files changed:**
- `components/custom-pricing-table.tsx` (lines 184–188)

---

### 4. ✅ Narrating Comment → Removed

**What was wrong:**
```ts
// Pre-select the suggested tier when first arriving at the pricing step
if (step === 3 && !selectedTier) {
  setSelectedTier(suggestedTier);
}
```
Comment just narrated the code; variable names already self-document.

**What was fixed:**
Comment removed; code stands on its own.

**Files changed:**
- `app/onboarding/page.tsx` (line 1315 onwards)

---

### 5. ✅ Unnecessary `useMemo` → Plain `const`

**What was wrong:**
```ts
const suggestedTier = useMemo(
  () => suggestTier({ role: selectedRole, goal: selectedGoal, treeCount }),
  [selectedRole, selectedGoal, treeCount]
);
```
`suggestTier()` is O(1) with minimal overhead. `useMemo` allocation + dependency tracking costs more than just calling the function.

**What was fixed:**
```ts
const suggestedTier = suggestTier({ role: selectedRole, goal: selectedGoal, treeCount });
```
Simple, direct, no performance loss.

**Files changed:**
- `app/onboarding/page.tsx` (line 289)
- Removed `useMemo` from React imports (line 15)

---

## PART II: TIER MESSAGING INCONSISTENCIES (5 ISSUES)

### MUST-FIX (2 issues)

#### Issue #1: Plus Tier Positioning Mismatch

**Problem:**
| Surface | Positioning | Risk |
|---------|-----------|------|
| **groundzy.com** | Optional add-on to Home ("Groundzy Plus" callout card) | Customer doesn't see Plus as a direct upgrade path |
| **auth.groundzy** | First-class tier (column in tier list: Home → Plus → Pro → Teams) | Plus is a full tier, not an extra feature |
| **app.groundzy** | First-class tier (`PlanTier = 'plus'`, pricing, capabilities) | When user downgrades from Pro, they should see Plus as option |

**Customer Impact:**
- Homeowner looking to upgrade from Home might skip Plus and go straight to Pro ($19 vs $9)
- No messaging clarifies that Plus is a stepping stone, not an add-on
- Checkout flow (auth.groundzy) offers Plus, but groundzy.com (where user first learned) never promoted it

**Severity:** MUST-FIX (revenue impact, UX confusion)

**Files to update:**
- `groundzy.com/index.html` — primary pricing/FAQ section (Plus appears as add-on in multiple blocks, e.g. ~lines 612, 713, structured data ~189–264)
- **Site-wide copy:** Other marketing pages repeat “add-on: Groundzy Plus” (e.g. `for-homeowners.html`, `blog/groundzy-free-for-homeowners.html`, `tree-mapping-software.html`, `blog/index.html`). Align or consciously exclude after the main `index.html` decision.
- `groundzy.com/modules/landing.js` — pricing toggle logic if it treats Plus separately
- Consider: Add a "Recommended upgrade path" visual on groundzy.com (Home → Plus → Pro → Teams)

---

#### Issue #2: Enterprise Tier Pricing Contradiction

**Problem:**
| Source | Claims | Reality |
|--------|--------|---------|
| **groundzy.com FAQ** | "For organizations with 51+ users… custom Enterprise pricing… Contact… at support@groundzy.com" (see FAQ ~788) | Explicit custom quote flow |
| **auth.groundzy** | No Enterprise tier in `PRICING_PLANS` (only Home, Plus, Pro, Teams) | Enterprise not offered at signup |
| **app.groundzy** | `lib/billing/tier-model.ts` lists `'Enterprise'` in `RUNTIME_TIERS` and teams grouping; `CHECKOUT_PLANS` is only `home` \| `plus` \| `pro` \| `teams` (no Enterprise at signup) | Enterprise exists in runtime typing, not in checkout keys |

**Customer Impact:**
- Customer reads "Enterprise = custom pricing" on website, tries to contact sales
- Auth system has no Enterprise tier to offer
- If Enterprise is truly custom, need a separate checkout flow; if it's Large Team rebranding, website copy is misleading

**Severity:** MUST-FIX (sales process broken, revenue risk)

**Decision needed:** Is Enterprise a real tier?

**If YES (separate custom pricing):**
- Add Enterprise to auth.groundzy `PRICING_PLANS` (no checkout, button → contact form)
- Update app.groundzy to handle Enterprise subscription status
- Add "Request Enterprise Quote" CTA to groundzy.com pricing

**If NO (Enterprise = Large Team rebranding):**
- Remove "Enterprise pricing is custom" from groundzy.com FAQ
- Update FAQ: "Large Team: $299/mo, 16–50 users. Contact us for 50+ seat custom arrangements."
- Remove `'Enterprise'` from app.groundzy `lib/billing/tier-model.ts` or mark it as deprecated (coordinate with Firestore / billing data that may already use the label)

**Recommendation:** Clarify intent with product team, then proceed with one of the above paths.

---

### NICE-TO-HAVE (3 issues)

#### Issue #3: Teams Tier Description — Minimal vs. Detailed

**Problem:**
| Source | Description |
|--------|-------------|
| **groundzy.com** | "Built for growing crews, municipalities, and teams with shared inventories" (specific, marketing-focused) |
| **auth.groundzy** | "For teams." (generic, minimal) |

**Customer Impact:** Low — both communicate "this is for teams", but groundzy.com's wording is more persuasive.

**Severity:** NICE-TO-HAVE (polish, not a blocker)

**Fix:** Update auth.groundzy `lib/pricing.ts` line 74:
```ts
description: "For growing crews, municipalities, and teams with shared inventories.",
```

---

#### Issue #4: Mid/Large Team Features — FAQ Claims vs. Enforced Capabilities

**Problem:**

groundzy.com FAQ (line 763) claims:
- *"Mid Team and above add API access and premium support."*
- *"Large Team adds white-label options and SLA guarantees."*

But `app.groundzy/lib/capabilities.ts` does NOT enforce these distinctions:
- API access: Not gated by Mid Team; no capability exists
- Premium support: Not explicitly gated (only "Priority" vs "Basic")
- White-label: No capability defined
- SLA: No capability defined

**Customer Impact:**
- Mid Team customer expects API access, finds none
- Large Team customer expects white-label, finds none
- Support team confused by FAQ promises not in product

**Severity:** NICE-TO-HAVE → escalate to product (this is a scope/roadmap issue, not a bug)

**Action:**
- If features are planned: Update auth.groundzy copy to remove premature claims (or move to "Roadmap")
- If features are not planned: Add them to capabilities.ts or remove from FAQ

---

#### Issue #5: Identifying Wand Spec — "1.2k Species" Marketing Detail

**Problem:**

groundzy.com (line 733): *"Our Identifying Wand can identify over 1.2k tree species with high accuracy."*

But `app.groundzy` and `auth.groundzy` do NOT enforce a species cap:
- `ai-usage.ts` tracks calls/month (10, 50, unlimited) — NOT species count
- No code references "1.2k species"

**Customer Impact:**
- Customer asks: "Can it really identify 1.2k species?" → unsupported claim
- Low impact (marketing copy, not a promise), but technical inconsistency

**Severity:** NICE-TO-HAVE (clarity, not a blocker)

**Action:**
- If "1.2k species" is a real spec: Add reference in auth.groundzy tier descriptions or add a note in `PRICING-REFERENCE.md`
- If it's approximate/unverified: Soften groundzy.com copy to "Identifies many tree species..." or remove the number

---

### Feature Enforcement Gap (2 LOW-PRIORITY issues)

#### Issue #6: API Access (Mid Team+) — Not in Capability System

**Status:** FAQ promises it, code doesn't enforce it.

**Action:** Add `"api.access"` to `app.groundzy/lib/capabilities.ts` and gate it by `isTierInGroup(tier, "teams") && tier !== "Small Team"`.

*Priority: LOW — escalate to product for API roadmap.*

---

#### Issue #7: White-Label & SLA (Large Team+) — Not in Capability System

**Status:** FAQ promises it, code doesn't enforce it.

**Action:** Add `"white_label"` and `"sla"` to `app.groundzy/lib/capabilities.ts` and gate by `tier === "Large Team" || tier === "Enterprise"`.

*Priority: LOW — escalate to product for roadmap.*

---

## PART III: FIX PLAN

### Phase 1: Clarify Business Rules (Week 1)

**Owner:** Product team
**Tasks:**
1. **Is Enterprise a real tier?** Decision tree:
   - YES → Add separate custom-quote flow to auth.groundzy
   - NO → Enterprise = Large Team rebranding; update FAQ

2. **Which features should Mid/Large unlock?**
   - API access?
   - White-label?
   - SLA?
   - Premium support? (already "Priority")

**Blockers:** Unless clarified, can't close Issues #2, #4, #6, #7.

---

### Phase 2: Messaging & Copy Alignment (Week 2–3)

**Owner:** Marketing (groundzy.com) + Product (messaging)

#### Task A: Fix Plus Tier Positioning

**Changes:**
1. **groundzy.com:** Update index.html pricing section
   - Move Plus from "add-on callout" to main tier comparison grid
   - Add visual/copy: "Recommended upgrade path: Home → Plus → Pro"
   - Line to change: ~612 (Home card); add Plus card alongside

2. **auth.groundzy:** Already correct (first-class tier in CustomPricingTable)

3. **Verify:** Test upgrade flows in app.groundzy (Home → Plus downgrade path should exist)

**Files to update:**
- `groundzy.com/index.html` (pricing section, ~lines 584–700)
- `groundzy.com/css/pricing.css` (if Plus card needs new styling)

---

#### Task B: Fix Enterprise Tier Messaging

**Once business decision is made** (Phase 1), implement one of:

**If Enterprise = Custom**
- auth.groundzy: Add "Enterprise" card to pricing table with "Contact us for custom quote" button
- groundzy.com: Keep current FAQ text but add link to custom quote form
- App.groundzy: Add Enterprise tier handling in subscription logic

**If Enterprise = Large Team**
- groundzy.com/index.html: Update FAQ block ~788 (and matching JSON-LD FAQ ~384) to remove "custom pricing" phrasing if Enterprise = Large Team
- New text: "For 50+ seats, contact our sales team at support@groundzy.com for volume arrangements."

**Files to update:**
- `groundzy.com/index.html` (FAQ section)
- `auth.groundzy/lib/pricing.ts` (if adding Enterprise tier)
- `app.groundzy/lib/capabilities.ts` (if need Enterprise logic)

---

#### Task C: Enrich Teams Description

**Minor update for marketing clarity:**

- `auth.groundzy/lib/pricing.ts` line 74: Change `"For teams."` to `"Built for growing crews, municipalities, and teams with shared inventories."`
- Verify i18n (Spanish, French) translations in `lib/i18n/messages.ts`

**Files to update:**
- `auth.groundzy/lib/pricing.ts`
- `auth.groundzy/lib/i18n/messages.ts` (ES, FR translations)

---

### Phase 3: Feature Roadmap Alignment (DEPENDS ON PHASE 1)

**Owner:** Product (roadmap decisions) + Engineering

**Tasks (only after Phase 1 business decisions):**

1. **If API access (Mid+) is committed:**
   - Add `"api.access"` capability to app.groundzy/lib/capabilities.ts
   - Gate: Mid Team, Large Team, Enterprise
   - Update auth.groundzy PRICING_FEATURE_KEYS to mention API in Mid+ feature list

2. **If White-label (Large) is committed:**
   - Add `"white_label"` capability
   - Gate: Large Team, Enterprise

3. **If SLA (Large) is committed:**
   - Add `"sla"` capability
   - Gate: Large Team, Enterprise

4. **If "1.2k species" spec is real:**
   - Document it in `docs/PRICING-REFERENCE.md`
   - Add reference in tier descriptions or FAQ
   - Keep groundzy.com copy as-is

**Files to update:**
- `app.groundzy/lib/capabilities.ts`
- `auth.groundzy/lib/pricing.ts` (PRICING_FEATURE_KEYS)
- `app.groundzy/docs/PRICING-REFERENCE.md`

---

### Phase 4: Testing & Verification (End of Phase 3)

**Owner:** QA + Product

**Checklist:**
- [ ] Signup flow offers Plus as a standalone tier with correct copy
- [ ] Pricing tables on groundzy.com and auth.groundzy show same information
- [ ] No contradictions between FAQ and product capabilities
- [ ] Enterprise flow (custom quote or Large Team rebranding) works end-to-end
- [ ] App.groundzy correctly enforces tier capabilities
- [ ] Analytics tracks Plus selections (ensure it's not getting skipped in upgrade flows)

---

## PART IV: Priority & Timeline

| Issue | Category | Priority | Phase | Effort | Owner |
|-------|----------|----------|-------|--------|-------|
| Code quality (5 fixed) | ✅ | DONE | — | DONE | ✅ Done |
| Plus positioning | Messaging | MUST | 2 | 1 day | Marketing |
| Enterprise contradiction | Messaging | MUST | 1 → 2 | 0.5 days + product | Product |
| Teams description | Copy | NICE | 2 | 15 min | Product |
| Mid/Large features | Scope | NICE | 1 → 3 | dependent | Product |
| Identifying Wand spec | Copy | NICE | 2 | 15 min | Product |
| API capability | Feature | LOW | 3 | 1 day | Engineering |
| White-label capability | Feature | LOW | 3 | 1 day | Engineering |
| SLA capability | Feature | LOW | 3 | 1 day | Engineering |

**Overall timeline: 1–2 weeks (depending on product decisions in Phase 1)**

---

## Summary Table: Issues → Fixes

| # | Issue | Severity | Status | Fix |
|---|-------|----------|--------|-----|
| 1 | Badge duplication | Code quality | ✅ DONE | Extracted `PricingBadge` component |
| 2 | Redundant `?? 0` | Code quality | ✅ DONE | Removed from onboarding-tier-suggestion.ts |
| 3 | Awkward badge condition | Code quality | ✅ DONE | Simplified to ternary |
| 4 | Narrating comment | Code quality | ✅ DONE | Removed |
| 5 | Unnecessary useMemo | Code quality | ✅ DONE | Changed to plain const |
| 6 | Plus tier positioning | Messaging | PLAN → FIX | Update groundzy.com to show Plus as first-class tier |
| 7 | Enterprise pricing contradiction | Messaging | PLAN → FIX | Clarify: custom or Large Team? Then align all surfaces |
| 8 | Teams description | Copy polish | NICE | Update auth.groundzy to match groundzy.com wording |
| 9 | Mid/Large features (API, SLA) | Scope gap | NICE | Add capabilities.ts logic if features are committed |
| 10 | Identifying Wand "1.2k species" | Copy clarity | NICE | Verify spec, update or remove from groundzy.com |

---

## Next Steps

1. **Executive decision:** Is Enterprise a real tier? (blocks Issues #2, #6, #7)
2. **Product roadmap:** Do Mid/Large teams get API, white-label, SLA? (blocks Issues #4, #9)
3. **Assign Phase 2 owners:** Marketing (Plus positioning), Product (copy), Engineering (capabilities)
4. **Track blockers** and ensure Phase 1 decisions unblock Phase 2–3 work
