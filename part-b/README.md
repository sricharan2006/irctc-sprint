# IRCTC Design Sprint — Part B: Feature Specifications & Roadmap

Welcome to Part B of the IRCTC design engineering sprint. This folder contains complete specifications, wireframes, technical plans, and prioritization for fixing the 6 critical problems discovered in Part A.

## 📋 Quick Start

**New to this project?** Start here:
1. Read [TRACEABILITY.md](TRACEABILITY.md) to understand how Part B solves Part A problems
2. Review [MATRIX.md](MATRIX.md) for the prioritized roadmap
3. Dive into [SPECS.md](SPECS.md) for detailed solutions
4. Check [WIREFRAMES.md](WIREFRAMES.md) to see UI mockups
5. Review [PEER-REVIEW.md](PEER-REVIEW.md) for key feedback & updates

---

## 📁 Document Structure

### Core Deliverables

| Document | Purpose | Audience |
|----------|---------|----------|
| [SPECS.md](SPECS.md) | 6 detailed feature specifications (Problem + Solution + Tech Plan + Metrics) | Engineers, PMs, Designers |
| [WIREFRAMES.md](WIREFRAMES.md) | Mid-fidelity UI mockups for all UI-related problems | Designers, Frontend engineers |
| [AI-FEATURE.md](AI-FEATURE.md) | Specific AI proposal (XGBoost waitlist predictor) | ML engineers, Data engineers, PMs |
| [MATRIX.md](MATRIX.md) | 2×2 Impact vs Effort matrix with prioritization | PMs, Leadership |
| [PEER-REVIEW.md](PEER-REVIEW.md) | Peer review feedback & spec updates | Entire team |
| [TRACEABILITY.md](TRACEABILITY.md) | Evidence chain: Part A problems → Part B solutions | PMs, QA, future reference |

---

## 🎯 The 6 Solutions at a Glance

### Phase 1: Core Booking Stability (Weeks 1–4)

**1️⃣ Tatkal Virtual Queue System** (Spec 1)
- **Solves:** Tatkal booking crashes at 10:00 AM
- **How:** Client-side queue + live countdown + 90-sec booking window
- **Impact:** 9/10 | **Effort:** 7/10 | **Status:** Priority #1
- **Expected outcome:** Completion rate 40% → 65%, HTTP 502 errors <1%

**3️⃣ Berth Selection State Persistence** (Spec 3)
- **Solves:** Berth preferences reset mid-booking
- **How:** Immutable state + backend validation + persistent banner
- **Impact:** 7/10 | **Effort:** 5/10 | **Status:** Priority #2 (parallel to #1)
- **Expected outcome:** Selection loss 35% → <2%, Mobile success 60% → 95%

### Phase 2: Discovery Improvements (Weeks 5–8)

**2️⃣ Persistent Search Filters** (Spec 2)
- **Solves:** Search filters reset after page refresh
- **How:** Client-side localStorage + Redux state + smart caching
- **Impact:** 7/10 | **Effort:** 5/10 | **Status:** Priority #3
- **Expected outcome:** Filter persistence 0% → 98%, Time saved 75 seconds/search

### Phase 3: Session & Trust Layers (Weeks 9–12)

**4️⃣ Persistent Session Management** (Spec 4)
- **Solves:** Repeated login prompts interrupt booking flow
- **How:** JWT token refresh + extended session (4 hours) + device trust
- **Impact:** 7/10 | **Effort:** 7/10 | **Status:** Priority #4
- **Expected outcome:** Login interruptions 22% → <1%

**5️⃣ PNR Clarity + AI Waitlist Predictor** (Spec 5 + AI-FEATURE)
- **Solves:** Cryptic PNR status messages, user anxiety about waitlist
- **How:** Error translation + XGBoost ML model (78–82% accuracy)
- **Impact:** 6/10 | **Effort:** 6/10 | **Status:** Priority #5 (parallel backend/frontend)
- **Expected outcome:** Support tickets <2%, Unnecessary rebookings <5%, User confidence 2.8/5 → 4.7/5

### Phase 4: Polish (Weeks 13+)

**6️⃣ Reservation Chart Redesign** (Spec 6)
- **Solves:** Cluttered chart layout, hard to find available berths
- **How:** Visual berth grid + color coding + coach carousel
- **Impact:** 5/10 | **Effort:** 8/10 | **Status:** Deferred (or incremental improvements only)
- **Note:** High effort, moderate impact. Recommended: Apply quick wins (color coding, mobile scrolling) until Phase 4

---

## 📊 Impact vs Effort Matrix

```
                    ↑ IMPACT
                    |
            10  ┌───┼────────────┐
                | 🏗 MAJOR       |
              8 │  1. Tatkal (9) │
                │  2. Filters(7) │
              6 │  3. Berth (7)  │ 4. Session(7) │  5. PNR (6)
                │                │
              2 │                │    6. Chart(5) → ❌ AVOID
                └────┼────────────┘─────→ EFFORT
                    0    5    8   10
```

**Quadrant Guide:**
- 🚀 **Quick Wins (top-left):** Not present (IRCTC problems are systemic)
- 🏗 **Major Projects (top-right):** Features 1–5 (high impact, high effort, all worth doing)
- 🧩 **Fill-Ins (bottom-left):** Not present (no low-effort, low-impact items identified)
- ❌ **Time Sinks (bottom-right):** Feature 6 (defer or incremental only)

---

## 🎨 Wireframe Summary

**UI mockups created for:**
1. Tatkal Queue (before/after/during/complete states)
2. Search Filters (persistence, cache metadata)
3. Berth Selection (persistent banner across steps)
4. Session Management (silent refresh vs expected login)
5. PNR Status (cryptic vs clear messages + prediction)
6. Reservation Chart (dense table vs visual grid)

**Format:** ASCII art + detailed annotations (Figma/design tool files can be created separately)

---

## 🤖 AI Feature Details

**Model:** XGBoost Classifier (Gradient Boosting)  
**Problem it solves:** Waitlisted users don't know if they'll get confirmed → causes panic rebooking  
**Input data:** Route, date, class, WL position, historical patterns  
**Training data:** 3+ years IRCTC booking history (15M+ records)  
**Output to user:** "Your WL #8 has 78% chance of confirmation by May 15"  
**Fallback:** If confidence <70%, show aggregate statistics ("70% of WL #8 on this route confirm")  

**Why XGBoost (not LLMs)?**
- Fast inference (<100ms vs LLM seconds)
- Interpretable (show feature importance to PM)
- Cost-effective (<$0.01 per prediction vs $0.20 for GPT)
- Works great with tabular data
- Proven in production ML systems

---

## ✅ Success Criteria

### By End of Phase 1 (Week 4)

- [ ] Tatkal queue system deployed to 5% canary
- [ ] Berth state persistence integrated into booking flow
- [ ] <1% booking state loss (vs current 8%)
- [ ] Zero payment flow regressions

### By End of Phase 2 (Week 8)

- [ ] Search filters persist across page refresh (98%+)
- [ ] Filter state synced across devices
- [ ] Result freshness improved (cache hit rate 60%+)

### By End of Phase 3 (Week 12)

- [ ] Session token refresh working silently (99.9% uptime)
- [ ] PNR error messages translated to user-friendly language
- [ ] AI model trained, tested, deployed (78%+ accuracy)
- [ ] All 5 features rolled out to 100% of users

### Overall Impact (Week 13+)

- [ ] User satisfaction: 3.4/5 → 4.6/5
- [ ] Support load reduced 40%
- [ ] Booking completion rate increased
- [ ] Platform stability improved (502 errors <1%)

---

## 🔗 Connecting to Part A

**Important:** Every spec in Part B traces back to specific evidence in Part A:

- **Feature 1** addresses [Part A Problem 1](../part-a/PROBLEMS.md#problem-1-tatkal-booking-crashes-at-1000-am-given)
- **Feature 2** addresses [Part A Problem 2](../part-a/PROBLEMS.md#problem-2-search-filters-do-not-work-reliably-given)
- **Feature 3** addresses [Part A Problem 3](../part-a/PROBLEMS.md#problem-3-seat-selection-resets-randomly-given)
- **Feature 4** addresses [Part A Problem 4](../part-a/PROBLEMS.md#problem-4-repeated-login-requirement-causes-booking-delays-self-discovered)
- **Feature 5** addresses [Part A Problem 5](../part-a/PROBLEMS.md#problem-5-correct-pnr-sometimes-shows-wrongerror-output-self-discovered)
- **Feature 6** addresses [Part A Problem 6](../part-a/PROBLEMS.md#problem-6-reservation-charts-are-cluttered-and-difficult-to-understand-self-discovered)

See [TRACEABILITY.md](TRACEABILITY.md) for detailed evidence chains.

---

## 📅 Implementation Timeline

```
Week 1-4:   Phase 1 — Tatkal Queue + Berth State Persistence
Week 5-8:   Phase 2 — Search Filters + Phase 1 Stabilization
Week 9-12:  Phase 3 — Session Mgmt + PNR Clarity + AI Model
Week 13+:   Phase 4 — Chart Redesign (if justified) + Optimization
```

**Team structure:**
- **Team A (6 eng):** Tatkal queue (Phase 1)
- **Team B (4 eng):** Berth state (Phase 1)
- **Team C (3 eng):** Search filters (Phase 2)
- **Teams A&B:** Session mgmt + PNR (Phase 3)
- **ML engineer:** AI training pipeline (Phase 3)

---

## 🛠 Technical Stack

### Frontend
- React (state management: Redux or Context API)
- Figma for detailed wireframes (link TBD)
- WebSocket/SSE for real-time updates (Tatkal queue, PNR status)

### Backend
- Node.js/Python Flask (existing IRCTC stack)
- PostgreSQL/MongoDB (existing DB)
- Redis (caching for queue, filters)
- XGBoost (AI model serving via Flask/FastAPI microservice)

### Infrastructure
- Kubernetes (queue service, AI service)
- FCM/OneSignal (push notifications)
- GitHub Actions (CI/CD)

---

## 📝 Key Documentation

**For PMs:**
- [MATRIX.md](MATRIX.md) — Prioritization rationale
- [TRACEABILITY.md](TRACEABILITY.md) — Problem-to-solution mapping

**For Engineers:**
- [SPECS.md](SPECS.md) — Technical implementation plans
- [AI-FEATURE.md](AI-FEATURE.md) — ML model details
- [PEER-REVIEW.md](PEER-REVIEW.md) — Design/arch decisions & rationale

**For Designers:**
- [WIREFRAMES.md](WIREFRAMES.md) — UI mockups & interactions
- [SPECS.md](SPECS.md) — Component requirements

**For QA:**
- Each spec's "Success Metrics" section — testable acceptance criteria
- [PEER-REVIEW.md](PEER-REVIEW.md) — Risk areas & edge cases

---

## ❓ FAQ

**Q: Why is chart redesign deferred?**  
A: Impact is moderate (5/10) but effort is very high (8/10). The other 5 features have higher ROI. We recommend incremental improvements (color coding, better mobile) until Phase 4. See [MATRIX.md](MATRIX.md) for rationale.

**Q: Can we build features 2–4 in parallel with phase 1?**  
A: Partially. Features 1 & 3 must start together (Phase 1). Feature 2 can start after Phase 1 reaches beta (Week 3–4). Features 4–5 start Week 9. See timeline in [MATRIX.md](MATRIX.md).

**Q: What if the AI model doesn't achieve 78% accuracy?**  
A: See [AI-FEATURE.md](AI-FEATURE.md#fallback--error-handling). If accuracy <75%, we rollback to aggregate statistics and retrain. User is not shown an inaccurate prediction.

**Q: How does this connect to Indian Railways backend?**  
A: Tatkal queue and berth systems integrate with existing Railways APIs. See [SPECS.md](SPECS.md) constraint: "Railway backend must support 5,000 bookings/min." This is a dependency to be coordinated pre-launch.

**Q: Who's responsible for peer review?**  
A: See [PEER-REVIEW.md](PEER-REVIEW.md). PM reviews top 2 specs; engineering lead reviews technical feasibility; design reviews wireframes. Async feedback compiled by Friday of review week.

---

## 🚀 Getting Started

### For Team Leads

1. Read [MATRIX.md](MATRIX.md) to understand priorities
2. Assign Phase 1 features to teams (Tatkal to Team A, Berth to Team B)
3. Schedule technical deep-dives on each spec
4. Set up estimation sessions for story points

### For Individual Engineers

1. Find your assigned spec in [SPECS.md](SPECS.md)
2. Review the "Technical Implementation Plan" section
3. Check [WIREFRAMES.md](WIREFRAMES.md) if UI-related
4. Review [PEER-REVIEW.md](PEER-REVIEW.md) for decisions & rationale
5. Add to your sprint board

### For Designers

1. Review [WIREFRAMES.md](WIREFRAMES.md) ASCII mockups
2. Create high-fidelity Figma designs for each wireframe
3. Add interaction annotations
4. Get feedback from design review group

### For QA

1. Extract success metrics from each spec
2. Write test cases for each metric
3. Identify edge cases in "Edge Cases & Constraints" section
4. Plan load testing for Features 1 & 5 (high-traffic areas)

---

## 📞 Questions?

- **Design decisions?** See [PEER-REVIEW.md](PEER-REVIEW.md)
- **Why solve problem X?** See [TRACEABILITY.md](TRACEABILITY.md)
- **What's the priority?** See [MATRIX.md](MATRIX.md)
- **How do we build it?** See [SPECS.md](SPECS.md)

---

## 📊 Document Stats

- **Total feature specs:** 6
- **Wireframes:** 6 (before/after comparisons)
- **API endpoints defined:** 15+
- **Success metrics:** 30+
- **Lines of specification:** 5,000+
- **Peer review feedback items:** 15+

---

## ✍️ Version History

| Date | Version | Changes |
|--|--|--|
| May 8, 2026 | 1.0 | Initial Part B complete (all 6 specs, wireframes, AI proposal, matrix, peer review) |

---

**Status:** ✅ Ready for Phase 1 kickoff (Week 1)

Last updated: May 8, 2026

