# IRCTC Impact vs Effort Matrix — Part B

## Overview

This document prioritizes all 6 solutions on a **2×2 matrix** based on:
- **Impact:** User reach, severity, frequency, financial consequence
- **Effort:** System complexity, number of components touched, risk, infrastructure changes

The matrix answers: **What should we build first? What's worth doing? What should we avoid?**

---

# Pre-Placement Scoring Table

Each solution is scored on Impact (1–10) and Effort (1–10), with detailed reasoning.

## Feature 1: Tatkal Virtual Queue System

**Impact Analysis:**
- **User reach:** 20–40 lakh users daily (peak hours 10:00–10:05 AM)
- **Frequency:** Daily, predictable window
- **Severity:** CRITICAL — platform crashes, bookings fail, payment uncertainty
- **Financial consequence:** Missed bookings = ruined trips, lost revenue for IRCTC, user frustration
- **Core flow:** Yes, this is the primary use case for many users

**Impact Score: 9/10** (Critical problem affecting highest-volume use case)

**Effort Analysis:**
- **Frontend changes:** New queue UI component (small, isolated)
- **Backend changes:** New queue service, state management (medium)
- **Database:** New `tatkal_queue` table, new indices (small)
- **Real-time layer:** WebSocket or SSE integration (medium complexity)
- **Third-party:** FCM/OneSignal integration for push notifications (small)
- **Payment gateway:** Modify booking timeout logic (medium risk)
- **Infrastructure:** May need Redis for queue caching if scale is extreme (medium)
- **Risk:** High — touching payment flow during critical booking window

**Effort Score: 7/10** (Complex, spans many systems, payment risk)

**Placement: HIGH IMPACT, HIGH EFFORT → Top Right Quadrant (Major Project)**

---

## Feature 2: Persistent Search Filters

**Impact Analysis:**
- **User reach:** All train search users (100% of IRCTC)
- **Frequency:** Every search session
- **Severity:** MEDIUM-HIGH — frustrating but not blocking (users can manually re-apply filters)
- **Financial consequence:** Increased search time = some users abandon before booking
- **Core flow:** Yes, central to discovery process

**Impact Score: 7/10** (Affects everyone, recurring frustration, blocks conversions)

**Effort Analysis:**
- **Frontend changes:** New Redux state manager for filters (small-medium)
- **Client-side caching:** Browser localStorage + session sync (small)
- **Backend changes:** Modified search endpoint to include cache metadata (small)
- **Database:** New `user_search_preferences` table, migration (small)
- **Cache layer:** Redis caching for search results (medium)
- **Real-time:** Cache refresh every 30 sec (small)
- **Risk:** Low — isolated change, no payment flow involved

**Effort Score: 5/10** (Moderate complexity, lower risk, well-isolated feature)

**Placement: HIGH IMPACT, MEDIUM EFFORT → Top Center (Major Project, but quicker than Tatkal)**

---

## Feature 3: Berth Selection State Persistence

**Impact Analysis:**
- **User reach:** 30–40% of booking users (those who care about berth choice)
- **Frequency:** Every completed booking in this segment
- **Severity:** MEDIUM-HIGH — breaks booking flow, forces restart
- **Financial consequence:** Booking abandonment, user frustration
- **Core flow:** Yes, within booking funnel

**Impact Score: 7/10** (Significant segment, completion blocker, high frustration)

**Effort Analysis:**
- **Frontend changes:** New React Context for berth state (small-medium)
- **Backend changes:** New berth reservation endpoint, validation logic (medium)
- **Database:** New `berth_selections` table with TTL (small-medium)
- **Session management:** Correlate berth reservation across booking steps (medium)
- **Real-time:** No real-time layer needed
- **Risk:** Medium — touches booking state machine, needs careful choreography

**Effort Score: 5/10** (Moderate complexity, contained scope, some state management risk)

**Placement: HIGH IMPACT, MEDIUM EFFORT → Top Center**

---

## Feature 4: Persistent Session Management & Auto-Renewal

**Impact Analysis:**
- **User reach:** Millions daily (everyone who browses multiple sessions)
- **Frequency:** Multiple times per session
- **Severity:** MEDIUM — frustrating interruption, not a deal-breaker
- **Financial consequence:** Lost time, some booking abandonment
- **Core flow:** Cross-cutting (affects all flows)

**Impact Score: 7/10** (Universal problem, affects decision-making, reduces friction)

**Effort Analysis:**
- **Frontend changes:** New session manager, token refresh interceptor (medium)
- **Backend changes:** JWT token management with refresh tokens (medium-high)
- **Database:** New `session_tokens` table, session management (medium)
- **Redis/cache:** Session store for fast lookups (small)
- **Security:** Token rotation, device fingerprinting (high complexity)
- **Risk:** HIGH — auth is security-sensitive, mistakes have large blast radius

**Effort Score: 7/10** (Complex, security-critical, spans frontend & backend)

**Placement: HIGH IMPACT, HIGH EFFORT → Top Right (Major Project, but risky)**

---

## Feature 5: PNR Enquiry Error Clarity & AI Prediction

**Impact Analysis:**
- **User reach:** Millions of PNR enquiry users, especially waitlisted passengers
- **Frequency:** Every PNR enquiry (multiple times per booking lifecycle)
- **Severity:** MEDIUM — causes support overhead, unnecessary rebookings, user anxiety
- **Financial consequence:** Support cost, opportunity cost (user anxiety), lost trust
- **Core flow:** Not core to booking, but affects post-booking experience

**Impact Score: 6/10** (Large reach but not blocking, high support impact)

**Effort Analysis (PNR Clarity component):**
- **Frontend changes:** PNR status translator module (small)
- **Backend changes:** Modify `/pnr/enquire` response format (small)
- **Database:** `pnr_status_translations` table with mappings (very small)
- **Risk:** Very low — isolated change, non-critical path

**Effort Score: 2/10** (For clarity component alone)

**Effort Analysis (with AI Predictor):**
- **Frontend:** UI to display prediction (small)
- **Backend:** Prediction service integration (medium)
- **ML model:** Training, validation, deployment (medium-high, one-time)
- **Data pipeline:** Historical data extraction, feature engineering (medium)
- **Infrastructure:** Container service for ML microservice (medium)
- **Risk:** Medium — ML model performance affects user trust

**Effort Score: 6/10** (For combined clarity + AI feature)

**Placement: MEDIUM IMPACT, MEDIUM EFFORT → Center (Fill-in or Major Project depending on phasing)**

---

## Feature 6: Reservation Chart Redesign

**Impact Analysis:**
- **User reach:** Thousands per reservation chart release, regular users
- **Frequency:** Once per user during booking
- **Severity:** MEDIUM-LOW — UX improvement, not a blocker
- **Financial consequence:** Minor (better UX doesn't directly impact bookings)
- **Core flow:** Part of booking journey (berth selection)

**Impact Score: 5/10** (Good UX improvement, affects discovery efficiency, not urgent)

**Effort Analysis:**
- **Frontend changes:** New React components for visual grid, carousel, touch interactions (large)
- **Backend changes:** New `/reservation-chart/visual` endpoint (small-medium)
- **Database:** `berth_visual_layout` configuration table (very small)
- **Real-time:** WebSocket updates for live berth changes (medium)
- **Mobile optimization:** Large redesign effort (high)
- **Accessibility:** WCAG AA compliance (medium)
- **Risk:** Medium — UI redesign has higher UX risk (users might hate new layout)

**Effort Score: 8/10** (Large frontend rework, mobile optimization, accessibility complexity)

**Placement: MEDIUM IMPACT, HIGH EFFORT → Bottom Right (Avoid or Defer)**

---

# Scoring Summary Table

| Feature | Problem | Impact | Effort | Quadrant | Priority |
|--|--|--|--|--|--|
| 1. Tatkal Queue | Crash at 10 AM | 9 | 7 | Major Project | 1 (do first) |
| 2. Search Filters | Filters reset | 7 | 5 | Major Project | 2 (do second) |
| 3. Berth State | Selection resets | 7 | 5 | Major Project | 2 (do second) |
| 4. Session Mgmt | Re-login prompts | 7 | 7 | Major Project | 3 (do after 1–3) |
| 5. PNR Clarity + AI | Unclear messages | 6 | 6 | Major Project | 4 (parallel track) |
| 6. Chart Redesign | Cluttered UI | 5 | 8 | Time Sink | 5 (defer) |

---

# The 2×2 Matrix

```
                    ↑ IMPACT
                    |
                    |
            10  ┌───┼──────────────────────┐
                |   │   MAJOR PROJECTS     │
                |   │   (Do these first)   │
              8 │   │                      │
                |   │  1. Tatkal ● (9,7)   │
                |   │  2. Filters ● (7,5)  │
              6 │   │  3. Berth ● (7,5)    │
                |   │  4. Session ● (7,7)  │
              4 │   │  5. PNR ● (6,6)      │
                |   │                      │
              2 │   │ ┌──────────────────┐ │
                |   │ │  QUICK WINS      │ │  FILL-INS
                |   │ │  (Do soon)       │ │  (Do when
              0 └───┼─┼──────────────────┼─┴──────→ EFFORT
                    0   2   4   6   8   10
                    
                6. Chart ●(5,8) → TIME SINK
                   (Avoid/Defer)
```

## Detailed Quadrant Explanations

### 🚀 Quick Wins (High Impact, Low Effort)

**None in this batch.** All solutions require moderate-to-high effort because IRCTC's problems are systemic (state management, real-time systems, payment flows).

**If we had one here:** Would be small UX fixes like "add success toast notification" or "improve error message wording."

### 🏗 Major Projects (High Impact, High Effort)

**All 5 solutions land here** (except Chart Redesign):

1. **Tatkal Virtual Queue (9 impact, 7 effort)** — Highest impact. Payment-critical. Build first.
2. **Search Filters (7 impact, 5 effort)** — High impact, lower effort. Build second (quick win among Major Projects).
3. **Berth State Persistence (7 impact, 5 effort)** — High impact, lower effort. Build second or parallel.
4. **Session Management (7 impact, 7 effort)** — High impact, high effort, security-sensitive. Build after core booking fixes.
5. **PNR Clarity + AI (6 impact, 6 effort)** — Solid impact. Can run as parallel track (backend focus on AI, frontend on messaging).

**Why they're worth it:** Each solves a million-user problem. Effort is justified by impact.

### 🧩 Fill-Ins (Low Impact, Low Effort)

**None identified.** This sprint tackled the biggest problems first.

### ❌ Time Sinks (Low Impact, High Effort)

**Reservation Chart Redesign (5 impact, 8 effort):**

This lands in the "Avoid" quadrant because:
- Impact is moderate (quality-of-life improvement, not a blocker)
- Effort is very high (entire frontend rework, mobile optimization, accessibility)
- Risk-reward: High risk (users might hate new interface) for moderate reward

**Recommendation:** Defer this until:
1. Core 5 problems are solved and stabilized
2. You have time/resources for a full design iteration
3. User research indicates frustration with current chart is limiting bookings

---

# Prioritization: The Implementation Roadmap

## Phase 1: Core Booking Flow Stability (Weeks 1–4)

**Goals:** Fix the most critical use case (Tatkal booking) and prevent state loss in booking flow.

**What to build:**
- ✅ Feature 1: Tatkal Virtual Queue (highest impact, critical)
- ✅ Feature 3: Berth State Persistence (prevents booking abandonment)

**Why this order:**
- Tatkal is the highest single-use-case volume (20–40 lakh daily)
- Berth state loss directly breaks the booking flow
- Both prevent failed bookings

**Team split:**
- Team A (Backend + Frontend): Tatkal queue system
- Team B (Backend + Frontend): Berth state manager

## Phase 2: Discovery & Browsing Improvements (Weeks 5–8)

**Goals:** Fix friction in the search and discovery phases.

**What to build:**
- ✅ Feature 2: Persistent Search Filters (high impact, moderate effort)

**Why this order:**
- After core booking works, improve discoverability
- Filters affect the browsing phase (earlier in funnel)
- Lower effort than session management, so quick win

**Team:**
- Team C (Frontend + Backend): Search filter persistence

## Phase 3: Session & Trust Layers (Weeks 9–12)

**Goals:** Reduce user friction and build trust in system reliability.

**What to build:**
- ✅ Feature 4: Session Management (high impact but security-sensitive)
- ✅ Feature 5: PNR Clarity + AI Predictor (parallel track)

**Why this order:**
- After core booking issues are resolved, focus on friction reduction
- Session management requires careful security review
- AI predictor can run in parallel (backend-heavy, different team)

**Team split:**
- Team A (Backend, Security): Session token management
- Team B (Backend, ML): AI training pipeline + PNR clarity messaging
- Team C (Frontend): PNR clarity UI + AI result display

## Phase 4: Polish & Optimization (Weeks 13+)

**Goals:** Improve UX and performance.

**What to defer:**
- ❌ Feature 6: Reservation Chart Redesign (low impact, high effort, high risk)

**Alternative:** Instead of full redesign, apply incremental improvements:
- Add color coding to current table (10% effort, 40% impact of full redesign)
- Improve mobile view with scrollable columns (20% effort)
- Add berth preview on hover (5% effort, 10% impact)

**When to do full redesign:** After Phase 1–3 stabilizes AND user feedback shows chart is limiting conversions.

---

# 3-Sentence Justifications for Each Placement

## Feature 1: Tatkal Virtual Queue → Major Project (Top Right)

The Tatkal use case represents 20–40 lakh daily users during a 5-minute window, making it IRCTC's single highest-priority problem; without queue management, the platform crashes and blocks the primary revenue-generating feature. Effort is substantial (spans frontend, backend, real-time layers, payment system) but justified by the scale of impact and reputational damage if Tatkal continues to fail. This is the top priority: fix it first or everything else is secondary.

---

## Feature 2: Search Filters → Major Project (Top Center)

Affecting 100% of IRCTC users and causing friction in the most important discovery phase, persistent filter state loss increases search time by 30–60% and contributes to booking abandonment. Effort is moderate (isolated Redux state management, client-side caching, small backend changes) with low security risk, making this a high-ROI project that can ship in 2–3 weeks. After Tatkal stability, this is the quickest high-impact win.

---

## Feature 3: Berth Selection Persistence → Major Project (Top Center)

Berth selection reset affects 30–40% of booking users and is a direct completion blocker (users restart entire flow or abandon), making it a critical funnel issue that directly impacts revenue. Effort is moderate (new state manager, berth reservation service, session choreography) with well-defined scope, enabling parallel development alongside other Phase 1 work. Combined with Tatkal fixes, this restores confidence in IRCTC's booking flow.

---

## Feature 4: Session Management → Major Project (Top Right)

Session-related login interruptions affect millions of users across all flows and erode trust in the platform's reliability; fixing this removes a systematic source of user frustration and reduces support load. Effort is high (JWT token architecture, security considerations, device fingerprinting) and touches critical auth paths, requiring careful implementation and thorough testing. This is a high-impact but security-sensitive project best tackled after core booking fixes stabilize.

---

## Feature 5: PNR Clarity + AI Predictor → Major Project (Center)

PNR enquiry clarity solves a widespread support burden (unclear status messages generate 12%+ of support tickets) and the AI prediction layer directly addresses user anxiety around waitlist confirmations, improving trust and reducing unnecessary rebookings. Effort is moderate-to-high due to the ML pipeline (training, validation, deployment) but can be built in parallel (backend team on ML, frontend team on messaging UI) without blocking core booking. This multi-layered solution justifies the effort by reducing support load and improving user confidence.

---

## Feature 6: Reservation Chart Redesign → Time Sink (Bottom Right)

While improved chart UX would enhance the berth selection experience, the impact is modest (quality-of-life improvement, not a blocker) and the effort is substantial (full frontend redesign, mobile optimization, accessibility compliance, high UX risk). Given the five higher-priority problems blocking bookings or causing system failures, this redesign should be deferred until Phase 4 and should only proceed if user research confirms the current chart is limiting conversion. Instead, apply incremental improvements (color coding, better mobile scrolling) for 20% effort.

---

# Cross-Feature Dependencies

```
Feature Dependencies & Recommended Build Order:

┌────────────────────────────────────────────────────────────┐
│ PHASE 1 (Weeks 1–4): Core Booking Stability               │
│                                                             │
│  Feature 1 (Tatkal Queue)                                  │
│  ├─ Depends on: Queue service, real-time layer             │
│  ├─ No blockers (standalone system)                        │
│  └─ START: Immediately (parallel to Feature 3)             │
│                                                             │
│  Feature 3 (Berth State Persistence)                       │
│  ├─ Depends on: Berth reservation API                      │
│  ├─ No blockers                                             │
│  └─ START: Immediately (parallel to Feature 1)             │
│                                                             │
├────────────────────────────────────────────────────────────┤
│ PHASE 2 (Weeks 5–8): Discovery Improvements               │
│                                                             │
│  Feature 2 (Search Filters)                                │
│  ├─ Depends on: Redux state management (internal)          │
│  ├─ Blocked by: None (Phase 1 independent)                 │
│  └─ START: When Feature 1 & 3 reach beta                   │
│                                                             │
├────────────────────────────────────────────────────────────┤
│ PHASE 3 (Weeks 9–12): Session & Trust Layers              │
│                                                             │
│  Feature 4 (Session Management)                            │
│  ├─ Depends on: Auth system                                │
│  ├─ Blocked by: None (but requires Phase 1 stability)      │
│  └─ START: Week 9 (can run with Features 5 & AI)           │
│                                                             │
│  Feature 5 (PNR Clarity + AI)                              │
│  ├─ Depends on: ML service, PNR backend                    │
│  ├─ Blocked by: None (independent)                         │
│  └─ START: Week 9 (parallel track)                         │
│                                                             │
├────────────────────────────────────────────────────────────┤
│ PHASE 4 (Weeks 13+): Polish                                │
│                                                             │
│  Feature 6 (Chart Redesign)                                │
│  ├─ Depends on: Full berth state (from Phase 1)            │
│  ├─ Blocked by: All Phase 1 must complete first            │
│  └─ START: Only if user research justifies (deferred)      │
│                                                             │
└────────────────────────────────────────────────────────────┘

Legend:
→ Serial dependency (do this after that completes)
// Parallel work (can do simultaneously)
⊗ No dependency (independent)
```

---

# Resource Allocation Recommendation

```
Team Composition & Timeline:

PHASE 1 (4 weeks):

Team A (6 engineers): Tatkal Queue
├─ 2 Backend (queue service, state management)
├─ 2 Frontend (queue UI, real-time updates)
├─ 1 DevOps/Infra (WebSocket setup, Redis, scaling)
└─ 1 QA (load testing, payment flow testing)

Team B (4 engineers): Berth State Persistence
├─ 2 Backend (berth reservation API, validation)
├─ 1 Frontend (berth state manager, UI updates)
└─ 1 QA (multi-step flow testing)

PHASE 2 (4 weeks):

Team C (3 engineers): Search Filters
├─ 1 Backend (filter cache, endpoint modification)
├─ 1 Frontend (Redux state, UI persistence)
└─ 1 QA (filter state testing)

Teams A & B (rotation): Stabilization & bug fixes from Phase 1

PHASE 3 (4 weeks):

Team A (4 engineers): Session Management
├─ 2 Backend (JWT tokens, device fingerprinting, security)
├─ 1 Frontend (token refresh interceptor)
└─ 1 QA/Security (security review, penetration testing)

Team B (4 engineers): PNR Clarity + AI
├─ 2 Backend (ML service, training pipeline)
├─ 1 Backend (PNR clarity integration)
├─ 1 Frontend (prediction display UI)
└─ 1 ML Engineer (model training, validation)

Team C (2 engineers): Support & incremental improvements

Total: ~25 engineers across 12 weeks
Parallel teams: Reduces risk & accelerates delivery
```

---

## Conclusion

This prioritization matrix ensures **maximum impact with managed resources.** The 6 solutions cluster into three clear tiers:

1. **Tier 1 (Weeks 1–4):** Fix critical booking failures (Tatkal, Berth State)
2. **Tier 2 (Weeks 5–8):** Reduce search friction (Filters)
3. **Tier 3 (Weeks 9–12):** Build trust layers (Session, PNR Clarity + AI)
4. **Tier 4 (Deferred):** Polish UX (Chart Redesign only if data supports it)

Following this roadmap **stabilizes IRCTC's core booking experience, eliminates systemic friction, and rebuilds user trust** within 12 weeks.

