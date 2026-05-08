# IRCTC Peer Review Session — Part B

## Session Summary

**Date:** May 8, 2026  
**Duration:** 90 minutes (5 min per spec × 2 specs + 40 min Q&A)  
**Attendees:**
- **Design Engineer** (You, presenting)
- **Product Manager** (PM, stakeholder)
- **Engineering Lead** (Backend, technical review)
- **Facilitator** (Note-taking, time management)

**Top 2 Specs Presented:**
1. **Feature Spec 1: Tatkal Virtual Queue System** (5 min)
2. **Feature Spec 5: PNR Clarity + AI Predictor** (5 min)

(Features 2–6 reviewed asynchronously due to time, notes in `ASYNC_REVIEW.md`)

---

# Session Notes: Feature 1 — Tatkal Virtual Queue

## Your Presentation (5 minutes)

> "IRCTC crashes during Tatkal booking, affecting 20–40 lakh users in a 5-minute window daily. The core issue: no queue management, leading to panic refreshes and cascading failures. Our solution: a client-side virtual queue that assigns users a position immediately, provides live countdown and progress tracking, gives them 90 seconds to complete booking when their turn arrives, then removes them from the queue. This prevents the request storm hitting the backend, gives users clear expectations, and dramatically improves booking completion rates. We've designed wireframes showing the queue UI at each stage—before opening, in queue, your turn, and completed. Technically, we need a new queue service, real-time layer, and modifications to the payment booking logic. Success metric: increase Tatkal completion rate from 40% to 75% during peak hours."

## PM Questions & Challenges

### Q1: "What if the queue predictor is completely wrong and users miss their window?"
**PM Concern:** If our algorithm says "you have 90 seconds" but actually they only have 60 before timeout, users lose their spot.

**Your Response (before review):**
- "The 90-second window is based on network latency testing and payment gateway limits."

**PM Challenge:**
- "That's based on *ideal* conditions. What about: (a) Users on 2G networks? (b) Slow payment gateway? (c) Payment gateway crashes mid-transaction?"

**Discussion Points Raised:**
- **On 2G networks:** 90 seconds might not be enough for form rendering + payment entry
- **Payment gateway SLA:** Do we have a guarantee from payment partner?
- **Failure recovery:** If user's 90 seconds expire mid-payment, can we recover without losing the queue position?

**Your Update (post-review):**
✅ **Added to Spec:** New "Edge Cases" section clarifying:
- On slow networks: Extend timer to 120 seconds (or detect network speed and adjust)
- Payment timeout: Don't reject immediately; check if payment eventually went through (3-min grace period)
- Partial payment: Reserve queue position for 5 minutes even if payment timed out (allow retry without losing spot)

---

### Q2: "This assumes Railway backend can handle the booking throughput. Can it?"

**PM Concern:** If the queue works perfectly but the Railway backend API can only process 500 bookings/min, users will still fail.

**Engineering Lead Challenge:**
- "We've never stress-tested the Railway API beyond 500 req/min. Tatkal could push it to 5000 req/min."
- "If Railway backend fails, who's responsible? We can't control that."

**Discussion Points Raised:**
- **Dependency risk:** IRCTC's queue is only useful if Railway backend can handle the load
- **SLA mismatch:** We need to coordinate with Railway tech team before launch
- **Fallback plan:** What if Railway API is slow? Do we queue locally until their system catches up?

**Your Update (post-review):**
✅ **Added to Spec:** New "Constraints & Assumptions" section:
- "Railway backend must support minimum 5,000 bookings/min (coordinated integration phase before launch)"
- "If Railway API response time exceeds 60 sec, queue system automatically pauses and sends alert to ops"
- "Fallback: If Railway API unavailable, queue is held and users notified with ETA to retry"

---

### Q3: "Won't users game the queue? What prevents queue-jumping?"

**Engineering Lead Concern:** Bad actors could manipulate the system to jump ahead in queue.

**Threat Examples Raised:**
- Network requests showing queue tokens—could users replay them?
- Could bots join queue 10,000 times?
- Could a single user create multiple accounts to get multiple queue slots?

**Discussion Points Raised:**
- **Security vulnerability:** Need rate limiting per user/device
- **Token security:** Queue tokens must be cryptographically signed
- **Bot detection:** IRCTC already has CAPTCHA; need to enforce on queue join

**Your Update (post-review):**
✅ **Added to Spec:** New "Security Considerations" section:
- "Queue tokens are JWT, cryptographically signed, single-use only"
- "Rate limiting: Max 3 queue joins per user per day (bypass only with support escalation)"
- "Bot detection: CAPTCHA required on queue join (same as current booking CAPTCHA)"
- "Device fingerprint: Queue position tied to device ID; if device changes, position revoked"

---

### Q4: "What's the actual booking completion rate target?"

**PM Question:** You said "increase from 40% to 75%". How confident are you in that number?

**Your Initial Response:**
- "Based on the assumption that if users aren't getting HTTP 502 errors, they'll complete more bookings."

**PM Challenge:**
- "That's correlation, not causation. Users might still abandon for other reasons (no available seats, wrong class, etc.)."
- "What if the real completion rate only goes to 55%? That's still good progress but below your target."
- "We need a more nuanced breakdown."

**Discussion Points Raised:**
- **Multiple completion blockers:** Queue system removes 502 errors but not seat unavailability, wrong pricing, etc.
- **Realistic target:** Maybe 40% → 60% is more defensible
- **Monitoring plan:** Need to track which step users abandon at

**Your Update (post-review):**
✅ **Updated in Spec:** Revised success metrics section:
- Primary metric: "Tatkal booking completion rate (from queue entry to PNR confirmation): 40% → 65%" (more realistic)
- Secondary metrics added:
  - "HTTP 502 errors during Tatkal: 8–12% → <1%"
  - "Booking abandonment at queue join step: 22% → <5%"
  - "Booking abandonment at berth selection step: (existing baseline) → (track separately)"
- Added metric: "User confidence in queue system: 0 → 80% agree 'I knew what was happening'"

---

## PM Summary Statement

> "This is a strong spec. The core idea (virtual queue) is sound and directly addresses the #1 platform crash. Your changes to handle network failures, Railway backend fallback, and security concerns make it deployable. Key action before engineering: coordinate with Railway tech team on backend capacity. I want to move this to the top of the roadmap—let's green-light Phase 1 starting with this."

---

# Session Notes: Feature 5 — PNR Clarity + AI Predictor

## Your Presentation (5 minutes)

> "PNR enquiry generates 12% of support tickets because users see cryptic backend messages like 'FLUSHED PNR' and don't understand what it means. We're solving this with two layers: (1) Translation layer that converts 'FLUSHED PNR' → 'Your ticket is confirmed on Indian Railways. Download from email.' (2) AI waitlist predictor that tells users 'Your WL #8 has 78% chance of confirmation 2 days before departure.' The predictor is an XGBoost model trained on 3+ years of booking data. We show the prediction immediately in the PNR result, along with recommended actions (set reminder, find alternatives, download ticket). If AI confidence is low (<70%), we show aggregate statistics instead. Success metrics: reduce PNR-related support tickets from 12% to 2%, reduce unnecessary rebookings from 18% to <5%, and increase user confidence in PNR status from 2.8/5 to 4.7/5."

## PM Questions & Challenges

### Q1: "What if the AI model is wrong at scale?"

**PM Concern:** XGBoost model validated on test data but fails in production. You tell 1 million users "Your WL will confirm" and it doesn't.

**Scenario Raised:**
- "It's May 15, 2026. Our model says 'WL #50 has 70% chance to confirm.' 50,000 users see this. By May 18, none of them confirmed."
- "Support gets flooded with angry users. Media coverage: 'IRCTC AI lied to users.'"
- "How do we recover from that?"

**Discussion Points Raised:**
- **Model risk:** ML models can fail on new data distributions
- **Calibration critical:** If model says 70% but only 30% actually confirm, users lose trust in *entire* platform
- **Monitoring plan:** What's our rollback strategy?

**Your Update (post-review):**
✅ **Added to Spec:** New "Monitoring & Validation" section:
- "Launch with 5% canary deployment (5% of users see predictions first)"
- "Daily validation: Compare predicted vs actual outcomes. If accuracy drops below 78%, alert ops"
- "If accuracy drops to <75%, rollback immediately to aggregate statistics (hide AI predictions)"
- "Monthly retraining: Monthly, retrain on 3 months of new data and validate against held-out test set"
- "Quarterly review: Executive review of model performance metrics"

---

### Q2: "How's the training data collected? Privacy concerns?"

**Engineering Lead Question:** Are we collecting individual PNR details to train the model?

**Your Initial Response:**
- "We use historical booking data from IRCTC database."

**Engineering Lead Pushback:**
- "That's vague. Does 'booking data' include passenger names? Phone numbers? Payment info?"
- "If we're using passenger PII to train the model, we have GDPR/privacy concerns."

**Discussion Points Raised:**
- **Data privacy:** What data is in the training set?
- **Regulatory compliance:** Do we need Privacy Board approval?
- **Data security:** How is 15M booking records handled during training?

**Your Update (post-review):**
✅ **Updated in Spec:** New "Data Privacy & Security" section:
- "Training data contains: route_id, date, class, wl_position, final_status ONLY"
- "Excludes: passenger names, phone, PNR number, payment info, email, etc."
- "All data is 3+ months old (aged-out, low privacy risk)"
- "Model inference includes no PII—only outputs: probability, confidence, estimated date"
- "Compliance: No new privacy concerns beyond existing booking data storage"

---

### Q3: "What if prediction is right but user behavior doesn't change?"

**PM Question:** Even if we accurately predict WL confirmation, does it actually change user behavior?

**Scenario Raised:**
- "User sees: 'Your WL has 80% chance.' User thinks: 'I don't trust that. I'll rebook anyway.'"
- "Success metric says 'reduce unnecessary rebookings.' But what if behavior doesn't shift?"

**Discussion Points Raised:**
- **User trust in AI:** Unproven. Users might distrust algorithmic predictions
- **Conservative users:** Even 80% confidence might not be enough for user to skip rebooking
- **Success metric risk:** "Reduce rebookings" assumes user believes the prediction

**Your Update (post-review):**
✅ **Added to Spec:** New "Measurement & Assumptions" section:
- "Assumption: Users with 78%+ confidence predictions will rebook 40% less than users without predictions"
- "Measurement approach: A/B test—50% of users see predictions, 50% don't. Compare rebooking rates"
- "Success criteria revised:"
  - Tier 1 (High confidence, 80%+): Rebooking rate should drop 40%
  - Tier 2 (Medium confidence, 60–80%): Rebooking rate should drop 20%
  - Tier 3 (Low confidence, <60%): Show aggregate stats, expect no behavior change"

---

### Q4: "Model deployment—who runs it? Where?"

**Engineering Lead Question:** This is a microservice. What's the operational plan?

**Concerns Raised:**
- "Where does the XGBoost model run? On IRCTC servers? Cloud?"
- "Who monitors it? ML engineer on-call 24/7?"
- "What if the service crashes? Is PNR enquiry still usable?"

**Discussion Points Raised:**
- **Operational complexity:** New service = new monitoring, alerts, scaling
- **Dependency:** If model service is down, does PNR enquiry fail?
- **Scaling:** Can the service handle 1M+ concurrent prediction requests during peak hours?

**Your Update (post-review):**
✅ **Added to Spec:** New "Operations & Infrastructure" section:
- "Deployment: Kubernetes pod (3 replicas for redundancy)"
- "Location: Co-hosted with main IRCTC API servers (same data center)"
- "Model file size: ~100 MB, loaded into memory on container startup"
- "Prediction latency: 50–200 ms (including network)"
- "Failover: If model service is down, API falls back to aggregate statistics (no error shown to user)"
- "Monitoring: CPU usage, memory, response latency, prediction accuracy (metrics dashboards)"
- "On-call: Backend engineer on rotation, escalate to ML engineer if model accuracy degradation detected"
- "Scaling: Horizontal scaling by replica count; capacity tested for 10,000 predictions/sec"

---

## PM Summary Statement

> "Strong spec. The AI idea is practical—XGBoost, not overselling with LLMs. Your clarifications on privacy, monitoring, and fallbacks address the big risks. I have one request: start with just the translation layer (no AI) for the first release. Ship that, get feedback, *then* add the AI predictor in release 2. This de-risks the launch and gives us data on whether users actually trust algorithmic predictions. Otherwise, looks good to green-light as Phase 3 work."

---

# Key Updates Summary

## Spec 1 Updates (Post-Review)

| Original | Updated | Reason |
|--|--|--|
| "90-second window" | "90–120 seconds based on network speed" | Handle 2G networks |
| No payment timeout recovery | Added 5-min grace period for payment validation | Handle payment latency |
| Assumption: "Railway backend can handle load" | Explicit SLA requirement + monitoring alert | De-risk integration |
| No security section | Added queue token signing, rate limiting, CAPTCHA, device fingerprint | Address gaming concerns |
| Completion rate target: "40% → 75%" | Revised: "40% → 65%" + secondary metrics | More realistic expectations |

---

## Spec 5 Updates (Post-Review)

| Original | Updated | Reason |
|--|--|--|
| "Use 3+ years of booking data" | Specified data fields (exclude PII) + privacy compliance | Address privacy concerns |
| Launch full AI immediately | Recommendation: Phase 1 (translation only), Phase 2 (add AI) | De-risk + get user feedback |
| Success metric: "reduce rebookings 18% → <5%" | Added A/B test plan + tiered expectations | Make measurement concrete |
| "Flask microservice" (vague) | Added: Kubernetes, 3 replicas, fallback logic, monitoring, on-call plan | Operational clarity |
| No model monitoring plan | Added: Accuracy threshold, rollback trigger, retraining schedule | Operational safety |

---

# Feedback That Did NOT Require Changes

These concerns were discussed but didn't necessitate spec updates (already addressed or trade-off accepted):

### Concern: "Tatkal queue might still not prevent crashes if Railway backend is overloaded"
**Response:** Acknowledged as external dependency. Our spec is sound; Railway coordination is an implementation task (Phase 0 before launch). PM agreed to own Railway sync.

### Concern: "Chart redesign (Feature 6) is higher priority than we thought"
**Response:** You held firm on matrix placement. Evidence: "Affects 5k users per release vs 40M for Tatkal." PM agreed: defer until Phase 4.

### Concern: "Search filters (Feature 2) might be easier than we estimated"
**Response:** Engineering Lead suggested effort score might be 4 instead of 5. You agreed to reassess during sprint planning. Matrix not changed (conservative estimate still valid).

---

# Peer Review Artifacts

## What Changed

✅ **Specs updated:** 2 out of 6 (top 2)  
✅ **Lines added:** ~300 (clarifications, new sections)  
✅ **Critical fixes:** 8 (edge cases, security, operations)  
✅ **Assumptions questioned:** 12  
✅ **New success metrics:** 7  
✅ **Implementation risks identified:** 6

## What's Next

1. **Async review (this week):** Features 2–6 get async PM + Engineering feedback (compiled by Friday)
2. **Roadmap finalization (next week):** Product team integrates peer review feedback and finalizes launch timeline
3. **Design sync (next week):** Wireframes presented to Design team for feedback on feasibility
4. **Engineering estimation (next week):** Each team estimates story points for assigned features
5. **Sprint planning (Week 3):** Kickoff Phase 1 with refined specs

---

# Facilitator Notes

**Key Themes from Discussion:**
1. **External dependencies matter:** Tatkal feature is only as good as Railway backend
2. **Data quality affects AI:** Privacy + data source clarity is non-negotiable for ML features
3. **Operations planning must come early:** New services (queue, ML) need ops support, not just engineering
4. **De-risk sequencing:** Sometimes "do translation layer before AI" is smarter than "do both at once"

**Recommendations for Next Sprint:**
1. PM to schedule Railway tech sync (coordinate backend capacity)
2. Data engineer to audit training data and document privacy compliance
3. DevOps to prepare Kubernetes config for ML service
4. Design to review wireframes with accessibility team

---

# Appendix: Async Review Summary (Features 2–6)

### Feature 2: Search Filters (PM + Engineering)
✅ **No changes required.** Spec is solid. Engineering Lead: "Can ship in 2 weeks." PM: "Clear ROI on filter persistence—good to go."

### Feature 3: Berth Selection Persistence (PM + Engineering)
✅ **1 update:** Add fallback logic for case where berth inventory changes between selection and booking. Spec updated.

### Feature 4: Session Management (Engineering + Security)
⚠️ **2 updates:** (1) Add device fingerprinting specifics (IP + User-Agent currently; need threat analysis), (2) Document token rotation schedule. Spec updated.

### Feature 5: (Covered in live session)

### Feature 6: Chart Redesign (Design + PM)
❌ **No updates.** Design Team: "Love the visual direction but implementation effort is 12+ weeks not 8. Recommend phased rollout." PM agreed to defer full redesign.

---

## Conclusion

**The peer review process validated 80% of specs, identified 8 critical gaps, and resulted in meaningful updates that make all 6 solutions more robust, realistic, and operationally sound.**

The team is now ready to move forward with Phase 1 (Tatkal Queue + Berth State Persistence) with confidence that edge cases, security, and operational requirements have been thought through.

