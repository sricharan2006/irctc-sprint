# IRCTC Design Sprint — Part B: Overview & Traceability

## Part B Scope

Part B transforms the 6 problems documented in Part A into **actionable feature specifications, technical plans, wireframes, AI proposals, and a prioritized roadmap.** Every deliverable traces back to evidence from Part A.

**Timeline:** 12 weeks (3 phases) to implement all 6 solutions  
**Team:** ~25 engineers across backend, frontend, ML, DevOps, QA, Security  
**Expected Impact:** 
- Tatkal completion rate: 40% → 65%  
- Session interruption reduction: 22% → <5%  
- Support tickets: 12% → <2%  
- Overall user satisfaction: 3.4/5 → 4.6/5  

---

# Part B Deliverables Checklist

| # | Deliverable | Location | Status |
|--|--|--|--|
| 1–6 | Feature Specs (6 problems) | [SPECS.md](SPECS.md) | ✅ Complete |
| 7 | Wireframes (UI mockups) | [WIREFRAMES.md](WIREFRAMES.md) | ✅ Complete |
| 8 | AI Feature Proposal | [AI-FEATURE.md](AI-FEATURE.md) | ✅ Complete |
| 9 | 2×2 Impact vs Effort Matrix | [MATRIX.md](MATRIX.md) | ✅ Complete |
| 10 | Peer Review Updates | [PEER-REVIEW.md](PEER-REVIEW.md) | ✅ Complete |
| 11 | Traceability Documentation | [TRACEABILITY.md](TRACEABILITY.md) (this file) | ✅ Complete |

---

# Evidence Chain: Part A → Part B

## Problem 1: Tatkal Booking Crashes at 10:00 AM

### Part A Evidence

**Location:** [PROBLEMS.md — Problem 1](../part-a/PROBLEMS.md#problem-1-tatkal-booking-crashes-at-1000-am-given)

**Key Facts:**
- **Affected users:** 20–40 lakh daily
- **Frequency:** Every day, 10:00–10:05 AM
- **Root cause:** "Platform cannot handle massive concurrent requests during Tatkal opening"
- **Symptoms:** HTTP 502 errors, CAPTCHA resets, session timeouts, payment uncertainty
- **Where it breaks:** Steps 6–9 (loading spinner, no feedback, server error)

### Part B Solution

**Location:** [SPECS.md — Feature Spec 1](SPECS.md#feature-spec-1-tatkal-virtual-queue-system)

**Solution Design:**
- **Address exact problem:** Client-side virtual queue prevents request storm by queuing users locally
- **Transparency:** Live countdown, position display, progress bar (solves "no feedback" issue)
- **State management:** Queue position persists; user given clear when their turn arrives (solves "payment uncertainty")
- **Fallback:** 90-second booking window; if timeout, user returns to queue tail without losing progress

**Success Metrics (tied to Part A):**
- "Tatkal booking completion rate (peak hours): 40% → 65%" ← Addresses "booking fail" problem
- "HTTP 502 errors during Tatkal: 8–12% → <1%" ← Addresses "server crashes" problem
- "Session timeout rate: 25% → <5%" ← Addresses "session timeout" symptom

**Wireframes:** [WIREFRAMES.md — Wireframe 1](WIREFRAMES.md#wireframe-1-tatkal-virtual-queue-screen)
- Before/After: Shows current crash state vs queue visibility
- Before state: Silent spinner, no position feedback → User panics
- After state: Queue position #4,281, "Your turn in 9 minutes" → User feels in control

**Impact & Effort:** [MATRIX.md — Feature 1](MATRIX.md#feature-1-tatkal-virtual-queue-system)
- Impact: 9/10 (highest-volume use case, critical problem)
- Effort: 7/10 (spans multiple systems, payment risk)
- **Priority: Phase 1, Week 1 (build first)**

---

## Problem 2: Search Filters Do Not Work Reliably

### Part A Evidence

**Location:** [PROBLEMS.md — Problem 2](../part-a/PROBLEMS.md#problem-2-search-filters-do-not-work-reliably-given)

**Key Facts:**
- **Affected users:** All train search users (100% of platform)
- **Frequency:** "Intermittently but frequently during high traffic periods"
- **Root cause:** "Filter state is not preserved properly and stale availability data appears after refreshes"
- **Symptoms:** Filters reset after refresh, waitlisted trains shown in "Available only" results
- **Where it breaks:** Steps 4–9 (page refresh, filter loss, manual search again)

### Part B Solution

**Location:** [SPECS.md — Feature Spec 2](SPECS.md#feature-spec-2-persistent-search-filters-with-smart-caching)

**Solution Design:**
- **Address exact problem:** Implement persistent filter state (client-side localStorage + backend sync)
- **Stale data fix:** Smart caching with 30-min TTL + real-time availability refresh
- **Cache metadata:** Show user "Last updated: 2 min ago" with manual refresh button
- **Cross-device sync:** Save filters to user account (sync across devices)

**Success Metrics (tied to Part A):**
- "Filter persistence across page refresh: 0% (resets) → 98%+" ← Solves filter reset
- "Stale availability data incidents: 8% → <1%" ← Solves "WL in Available" issue
- "Time to find train (with filters): 120 sec → 45 sec" ← Faster discovery

**Wireframes:** [WIREFRAMES.md — Wireframe 2](WIREFRAMES.md#wireframe-2-search-filters---persistent-state)
- Before/After: Shows filter reset on refresh
- Before: Filters gone after refresh, WL trains now visible (frustrating)
- After: Filters persist, results cached and fresh (clear expectation)

**Impact & Effort:** [MATRIX.md — Feature 2](MATRIX.md#feature-2-persistent-search-filters)
- Impact: 7/10 (affects 100% of users, recurring frustration)
- Effort: 5/10 (isolated feature, low risk)
- **Priority: Phase 2, Week 5 (high-ROI quick win)**

---

## Problem 3: Seat Selection Resets Randomly

### Part A Evidence

**Location:** [PROBLEMS.md — Problem 3](../part-a/PROBLEMS.md#problem-3-seat-selection-resets-randomly-given)

**Key Facts:**
- **Affected users:** 30–40% of booking users (those who care about berth)
- **Frequency:** "Frequently during booking flows involving seat selection"
- **Higher on mobile:** 60% on desktop succeed, 40% on mobile (network transitions)
- **Root cause:** "Seat selection state is not preserved correctly between booking flow components"
- **Symptoms:** Berth preference changes to "Auto", user forced to restart
- **Where it breaks:** Steps 5–7 (navigate to passenger details, preference resets)

### Part B Solution

**Location:** [SPECS.md — Feature Spec 3](SPECS.md#feature-spec-3-berth-preference-state-persistence)

**Solution Design:**
- **Address exact problem:** Immutable berth selection state (persist across entire booking flow)
- **Backend validation:** Reserve berth preference immediately, validate on each step
- **Persistent banner:** Show "Your seat: Lower Berth, Window Side" on every screen
- **Expiry management:** Reservation expires after 15 minutes if user abandons flow

**Success Metrics (tied to Part A):**
- "Berth selection loss during booking: 35% → <2%" ← Solves "resets" problem
- "Mobile berth selection success rate: 60% → 95%+" ← Solves mobile network issue
- "Elderly user (60+) berth success: 55% → 85%+" ← Addresses accessibility

**Wireframes:** [WIREFRAMES.md — Wireframe 3](WIREFRAMES.md#wireframe-3-berth-selection-state-persistence)
- Before/After: Shows berth reset mid-flow
- Before: Lower selected → Auto assigned (confusion)
- After: "Your seat: Lower Berth" persists across steps (clear, reliable)

**Impact & Effort:** [MATRIX.md — Feature 3](MATRIX.md#feature-3-berth-selection-state-persistence)
- Impact: 7/10 (completion blocker for 30–40% of users)
- Effort: 5/10 (state management, contained scope)
- **Priority: Phase 1, Week 1 (parallel to Tatkal Queue)**

---

## Problem 4: Repeated Login Requirement Causes Booking Delays

### Part A Evidence

**Location:** [PROBLEMS.md — Problem 4](../part-a/PROBLEMS.md#problem-4-repeated-login-requirement-causes-booking-delays-self-discovered)

**Key Facts:**
- **Affected users:** Millions daily during active sessions
- **Frequency:** "Multiple times during session" (navigation between modules)
- **Root cause:** "Platform fails to maintain stable authentication sessions across multiple IRCTC modules"
- **Symptoms:** Login popups interrupt flow, CAPTCHA resets, user loses progress
- **Where it breaks:** Step 5 (navigation to reservation charts, unexpected login prompt)

### Part B Solution

**Location:** [SPECS.md — Feature Spec 4](SPECS.md#feature-spec-4-persistent-session-management--auto-renewal)

**Solution Design:**
- **Address exact problem:** Automatic token refresh before expiry (silent, no UI interruption)
- **Extended session:** 4-hour lifetime (vs current 30 min)
- **Refresh tokens:** Long-lived (30 days), used to silently renew access tokens
- **Device trust:** Remember device, reduce re-authentication friction

**Success Metrics (tied to Part A):**
- "Unexpected login prompts during booking: 22% → <1%" ← Solves interruption problem
- "Session timeout rate: 18% → <2%" ← Solves premature expiry
- "Avg. time from login to confirmed booking: 240 sec → 180 sec" ← Faster booking

**Wireframes:** [WIREFRAMES.md — Wireframe 4](WIREFRAMES.md#wireframe-4-session-persistence--auto-login)
- Before/After: Shows unexpected login popup
- Before: User browsing 50 min → session expires → login popup (frustration)
- After: User browsing, token silently refreshes, no interruption (seamless)

**Impact & Effort:** [MATRIX.md — Feature 4](MATRIX.md#feature-4-persistent-session-management--auto-renewal)
- Impact: 7/10 (universal frustration, affects decision-making)
- Effort: 7/10 (auth is security-sensitive, spans frontend & backend)
- **Priority: Phase 3, Week 9 (after core booking stabilizes)**

---

## Problem 5: Correct PNR Sometimes Shows Wrong/Error Output

### Part A Evidence

**Location:** [PROBLEMS.md — Problem 5](../part-a/PROBLEMS.md#problem-5-correct-pnr-sometimes-shows-wrongerror-output-self-discovered)

**Key Facts:**
- **Affected users:** Millions of PNR enquiry users daily
- **Frequency:** "Intermittently depending on server sync delays"
- **Root cause:** "System exposes technical backend messages instead of user-friendly status explanations"
- **Symptoms:** "FLUSHED PNR", "PNR NOT YET GENERATED" (no explanation)
- **Consequence:** User confusion, unnecessary support calls, panic rebooking

### Part B Solution

**Location:** [SPECS.md — Feature Spec 5](SPECS.md#feature-spec-5-pnr-enquiry-error-clarity--smart-status-prediction) & [AI-FEATURE.md](AI-FEATURE.md)

**Solution Design:**
- **Part 1 — Error Clarity:** Translate backend codes to user-friendly messages (e.g., "FLUSHED PNR" → "Your ticket is confirmed on Indian Railways")
- **Part 2 — AI Predictor:** For waitlisted PNRs, show data-backed probability ("Your WL #8 has 78% chance of confirmation")
- **Recommended actions:** "Check again May 15", "Set reminder", "Find alternatives"

**Success Metrics (tied to Part A):**
- "Support tickets for 'unclear PNR status': 15% → <2%" ← Solves confusion
- "Unnecessary rebookings due to unclear status: 18% → <2%" ← Solves panic rebooking
- "PNR enquiry user satisfaction: 2.5/5 → 4.7/5" ← Solves trust issue

**Wireframes:** [WIREFRAMES.md — Wireframe 5](WIREFRAMES.md#wireframe-5-pnr-enquiry---error-clarity)
- Before/After: Shows cryptic message vs clear explanation
- Before: "FLUSHED PNR" (user confused, panic-calls support)
- After: "Your ticket is confirmed. Download from email." (user confident, no call needed)

**AI Details:** [AI-FEATURE.md — Waitlist Confirmation Predictor](AI-FEATURE.md)
- Model: XGBoost classifier (trained on 3+ years booking data)
- Input: Route, date, class, WL position
- Output: Probability (e.g., 78%), confidence level, estimated confirmation date
- Fallback: If confidence <70%, show aggregate statistics (no risky prediction)

**Impact & Effort:** [MATRIX.md — Feature 5](MATRIX.md#feature-5-pnr-enquiry-error-clarity--ai-predictor)
- Impact: 6/10 (large reach, high support impact, not blocking)
- Effort: 6/10 (translation simple, AI training complex)
- **Priority: Phase 3, Week 9 (can run parallel track)**

---

## Problem 6: Reservation Charts Are Cluttered and Difficult to Understand

### Part A Evidence

**Location:** [PROBLEMS.md — Problem 6](../part-a/PROBLEMS.md#problem-6-reservation-charts-are-cluttered-and-difficult-to-understand-self-discovered)

**Key Facts:**
- **Affected users:** Thousands per reservation chart release (elderly, mobile users, first-time users)
- **Frequency:** Every chart viewing (always reproducible)
- **Root cause:** "Interface lacks proper information hierarchy, visual grouping, color differentiation"
- **Symptoms:** Dense table, hard to scan, mobile unreadable, users miss available berths
- **Where it breaks:** Steps 4–7 (dense columns, repeated manual inspection, user spends extra time)

### Part B Solution

**Location:** [SPECS.md — Feature Spec 6](SPECS.md#feature-spec-6-redesigned-reservation-chart-with-information-hierarchy)

**Solution Design:**
- **Address exact problem:** Replace dense table with visual berth grid
- **Color-coded status:** Green (available), red (booked), grey (blocked)
- **Coach carousel:** Navigate coaches without scrolling
- **Touch-friendly:** Large berth tiles (50×50px on mobile)
- **Info hierarchy:** Berth #, type, price, side immediately visible

**Success Metrics (tied to Part A):**
- "Time to find available berth: 120 sec → 20 sec" ← Faster scanning
- "Elderly user chart comprehension: 35% → 90%+" ← Solves accessibility
- "Mobile user satisfaction: 2.1/5 → 4.6/5" ← Solves mobile readability

**Wireframes:** [WIREFRAMES.md — Wireframe 6](WIREFRAMES.md#wireframe-6-reservation-chart---visual-redesign)
- Before/After: Shows dense table vs visual grid
- Before: Dense table, hard to scan, "Where's an available berth?"
- After: Color-coded grid, "Available berths: 5, 7, 8" (instant clarity)

**Impact & Effort:** [MATRIX.md — Feature 6](MATRIX.md#feature-6-reservation-chart-redesign)
- Impact: 5/10 (good UX improvement, not urgent or blocking)
- Effort: 8/10 (large frontend rework, mobile optimization)
- **Priority: Phase 4, Week 13+ (defer, do incremental improvements first)**

---

# Cross-Problem Themes

### Problem 1 & 3: State Management Failures
**Root Cause (both):** System fails to preserve state across multi-step flows and page navigations  
**Common Pattern:** User's choice (berth selection, queue position) gets lost when moving to next step  
**Solution Approach:** Immutable state + backend validation + persistent UI confirmation  

### Problem 2 & 4: Session & Context Loss
**Root Cause (both):** User context (filters, authentication) not maintained across browsing  
**Common Pattern:** User navigates away, returns, loses previous context  
**Solution Approach:** Client-side caching + server-side sync + token refresh  

### Problem 5 & 6: Information Clarity Issues
**Root Cause (both):** System shows technical/dense information without user context  
**Common Pattern:** User doesn't understand what's happening, cannot find what they need  
**Solution Approach:** Translation + visual hierarchy + actionable guidance  

---

# Verification: All Part A Problems Are Addressed

| Problem | Part A Ref | Part B Solution | Spec Location | Status |
|--|--|--|--|--|
| 1. Tatkal Crash | [Link](../part-a/PROBLEMS.md#problem-1-tatkal-booking-crashes-at-1000-am-given) | Virtual Queue System | [SPECS.md](SPECS.md#feature-spec-1-tatkal-virtual-queue-system) | ✅ |
| 2. Filter Reset | [Link](../part-a/PROBLEMS.md#problem-2-search-filters-do-not-work-reliably-given) | Persistent Filters + Caching | [SPECS.md](SPECS.md#feature-spec-2-persistent-search-filters-with-smart-caching) | ✅ |
| 3. Berth Reset | [Link](../part-a/PROBLEMS.md#problem-3-seat-selection-resets-randomly-given) | State Persistence | [SPECS.md](SPECS.md#feature-spec-3-berth-preference-state-persistence) | ✅ |
| 4. Login Interruption | [Link](../part-a/PROBLEMS.md#problem-4-repeated-login-requirement-causes-booking-delays-self-discovered) | Session Auto-Renewal | [SPECS.md](SPECS.md#feature-spec-4-persistent-session-management--auto-renewal) | ✅ |
| 5. PNR Clarity | [Link](../part-a/PROBLEMS.md#problem-5-correct-pnr-sometimes-shows-wrongerror-output-self-discovered) | Error Translation + AI Predictor | [SPECS.md](SPECS.md#feature-spec-5-pnr-enquiry-error-clarity--smart-status-prediction) & [AI-FEATURE.md](AI-FEATURE.md) | ✅ |
| 6. Chart Cluttered | [Link](../part-a/PROBLEMS.md#problem-6-reservation-charts-are-cluttered-and-difficult-to-understand-self-discovered) | Visual Redesign | [SPECS.md](SPECS.md#feature-spec-6-redesigned-reservation-chart-with-information-hierarchy) | ✅ |

---

# Metrics Alignment: Part A → Part B → Success

**How each Part B metric traces back to Part A problem severity:**

### Tatkal (Problem 1 → Spec 1)
- **Part A frequency:** "Daily, 20–40 lakh users" → Part B metric: "Affects 40M users per quarter"
- **Part A severity:** "Critical blocker (HTTP 502)" → Part B metric: "502 errors: 8–12% → <1%"
- **Part A impact:** "Ruins trips, loses revenue" → Part B metric: "Completion rate: 40% → 65%"

### Search Filters (Problem 2 → Spec 2)
- **Part A frequency:** "Every search session" → Part B metric: "Affects 100% of users"
- **Part A severity:** "Medium frustration" → Part B metric: "Time cost: 120 → 45 sec"
- **Part A impact:** "Some abandonment" → Part B metric: "Conversion improvement measured"

### Session Management (Problem 4 → Spec 4)
- **Part A frequency:** "Multiple times per session" → Part B metric: "Login prompts: 22% → <1%"
- **Part A severity:** "Workflow interruption" → Part B metric: "Booking time: 240 → 180 sec"
- **Part A user segment:** "Mobile, unstable internet" → Part B metric: "Mobile re-auth: 35% → <5%"

---

# How to Use This Traceability

1. **For Product Managers:** Use to justify roadmap priorities to executives ("We're solving Problem 1 which affects 40M users quarterly")
2. **For Engineers:** Use to understand *why* each technical requirement exists (e.g., "90-sec timeout exists because Tatkal booking needs to complete before payment gateway times out")
3. **For Designers:** Use to validate wireframes against original user pain ("Chat grid fixes Problem 6 because users couldn't scan rows")
4. **For QA:** Use to write test cases that verify original problem is solved (e.g., "Test filter persistence after browser refresh—this was Problem 2")
5. **For future sprints:** Reference this doc if similar issues resurface ("Berth state loss happened again? Check Spec 3 edge cases")

---

# Next Steps: From Specs to Implementation

## Week 1: Finalizing Specs (Post-Peer-Review)

- ✅ Integrate peer review feedback into all specs (completed)
- ✅ Wireframes reviewed with design team
- ⏳ Engineering estimation for each feature (story points, weeks)
- ⏳ Data engineer audits training data for AI model (privacy compliance)
- ⏳ DevOps prepares infrastructure (Kubernetes, Redis, etc.)

## Week 2: Sprint Planning (Phase 1)

- ⏳ Assign features to teams
- ⏳ Break down specs into stories/tasks
- ⏳ Define acceptance criteria (linked to success metrics in specs)
- ⏳ Identify blockers (e.g., Railway backend coordination for Tatkal)

## Week 3: Phase 1 Launch

- ⏳ **Team A starts:** Tatkal Virtual Queue (Feature 1)
- ⏳ **Team B starts:** Berth State Persistence (Feature 3)
- ⏳ Daily standup, sprint velocity tracking

---

# Success Criteria for Part B

✅ **Specs are complete:** All 6 features have Problem + Solution + Tech Plan + Metrics + Edge Cases  
✅ **Wireframes are detailed:** Before/After comparison, component labels, interaction notes  
✅ **AI is specific:** Named model (XGBoost), sourced data (3+ years), output defined (probability + date)  
✅ **Matrix is justified:** All 6 solutions scored and placed with 3-sentence rationale  
✅ **Peer review is documented:** Feedback integrated, assumptions questioned, risks identified  
✅ **Traceability is clear:** Every spec references specific Part A problem with evidence  

---

# Summary

Part B successfully transforms Part A's 6 discovered problems into a **concrete, prioritized, peer-reviewed roadmap** with detailed specifications, wireframes, technical plans, and success metrics. Every deliverable is grounded in Part A evidence and evidence chain is clear.

The next phase is implementation (Phase 1 starts Week 3).

