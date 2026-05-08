# IRCTC AI Feature Proposal — Part B

## Executive Summary

This document proposes a **Waitlist Confirmation Probability Predictor** — an AI model that predicts whether a waitlisted ticket will be confirmed before the journey date. This feature directly solves **Problem 5 (PNR Status Clarity)** by transforming vague "waitlist" status into actionable, confidence-backed predictions.

The model is technically grounded (XGBoost classifier), data-grounded (3+ years of railway booking history), and deployable within existing IRCTC infrastructure (Flask/FastAPI microservice). It reduces user anxiety, prevents unnecessary rebookings, and improves trust in the PNR enquiry system.

---

# The Problem This AI Feature Solves

**Part A Traceability:** [Problem 5: Correct PNR Sometimes Shows Wrong/Error Output](../part-a/PROBLEMS.md#problem-5-correct-pnr-sometimes-shows-wrongerror-output-self-discovered)

**Problem Statement (expanded from Part A):**

When users book waitlisted tickets on IRCTC, they have no information about their actual chances of confirmation. A user with WL #8 on a popular route doesn't know if:
- This position usually confirms (high-demand route)?
- This position usually doesn't confirm (off-peak route)?
- How many days before departure confirmation typically happens?

Result: Users panic and take expensive actions:
1. Book backup tickets on competing operators
2. Rebook same train in different class
3. Choose different travel dates
4. Call support unnecessarily (increasing support load)
5. Travel with high anxiety instead of confidence

**Evidence from Part A:**
- Affected users: Lakhs of daily PNR enquiry users, especially waitlisted passengers
- Frequency: Every time a user checks their waitlisted PNR
- Current experience: Cryptic message "PNR NOT YET GENERATED" or unclear confirmation status
- Consequence: Wasted money, wasted support time, user anxiety

**This AI feature removes the anxiety** by providing a data-backed probability: "Your WL #8 has a 78% chance of confirmation 2 days before departure" based on 3+ years of historical patterns.

---

# AI Model Choice: Why XGBoost?

## Selected Model: XGBoost Classifier

**What it is:** Gradient Boosting Decision Tree ensemble. Industry-standard for tabular data prediction in production systems (used by Amazon, Airbnb, DoorDash).

**Why XGBoost (not alternatives):**

| Alternative | Why Not This One |
|--|--|
| **Neural Network (Deep Learning)** | Overkill for tabular booking data. Requires 100M+ rows to outperform XGBoost. IRCTC doesn't have that scale in clean format. Slower inference (milliseconds vs microseconds). Hard to explain predictions to product team. |
| **Logistic Regression** | Too simple. Cannot capture non-linear patterns (e.g., "WL #8 on route X in December != WL #8 on route X in June"). |
| **Random Forest** | Works but slower than XGBoost. XGBoost has superior regularization to prevent overfitting. |
| **Rule-based system** | No, too brittle. Hand-written rules like "WL < 10 = confirm" fail on edge cases. Data-driven model adapts. |
| **LLM (GPT-4)** | Completely inappropriate. These models don't work well for tabular prediction. Cost prohibitive (₹200+ per prediction vs ₹0.01 for XGBoost). Unexplainable. Unnecessary complexity. |

**Why XGBoost specifically:**
- ✅ Fast inference: 1–5 milliseconds per prediction (serves real-time on every PNR enquiry)
- ✅ Explainable: Can extract feature importance to understand what drives predictions
- ✅ Robust: Handles missing data, imbalanced classes (confirmations are < cancellations)
- ✅ Proven: Already used in railway industry (MakeMyTrip, Goibibo prediction models)
- ✅ Deployable: Lightweight model file (50–100 MB), runs on basic hardware
- ✅ Cost-effective: No per-inference costs like cloud ML APIs

---

# Training Data & Data Requirements

## Data Source: Historical IRCTC Booking Database

**Time period:** Jan 2023 – May 2026 (3.5 years)
**Records:** ~15–20 million booking transactions
**Focus:** Sleeper and AC2 classes (covers 85% of IRCTC volume)

### Training Features (Input Variables)

**Route-level features:**
- `route_id` (e.g., Delhi-Chennai route)
- `route_popularity_score` (historical booking volume for this route)
- `route_avg_confirmation_rate` (what % of WL tickets confirm on this route?)

**Temporal features:**
- `travel_date` (day of week: 1–7)
- `day_of_month` (1–31, captures month-end vs month-start patterns)
- `week_of_year` (1–52, captures seasonal demand)
- `is_holiday` (binary: festival, vacation period, public holiday?)
- `is_school_break` (binary: summer vacation, winter break)
- `days_until_departure` (1–90, how many days away is the journey?)

**Class & quota features:**
- `class_type` (Sleeper, AC2, AC3, FC)
- `quota_type` (General, Tatkal, Premium, Senior Citizen)
- `reservation_quota_availability` (enum: available_quota, restricted_quota, no_quota)

**Waitlist position features:**
- `wl_position_at_booking` (1–5000, user's position when they booked)
- `wl_position_2_days_before` (1–5000, position 2 days before departure, if available)
- `wl_position_7_days_before` (1–5000, position 7 days before departure, if available)

**Historical outcome features:**
- `historical_booking_count_for_route_date_class` (how many people booked this route/date/class combo historically?)
- `historical_cancellation_rate` (what % usually cancel?)
- `historical_avg_confirmation_day` (on average, WL confirmations happen X days before departure)

**Target Variable:**
- `final_status` (binary: 1 = Confirmed, 0 = Not Confirmed)

### Data Quality & Preprocessing

**Assumptions:**
- Clean, deduplicated booking records from IRCTC master database
- No privacy concerns: aggregate historical patterns (not individual passenger data)

**Handling missing data:**
- `wl_position_7_days_before`: If PNR cancelled before this checkpoint, mark as missing
- Missing values → imputed with route averages (e.g., "typical cancellation rate for this route")

**Class imbalance:**
- Confirmed WL bookings: ~70% of dataset
- Not confirmed WL bookings: ~30% of dataset
- Use XGBoost's `scale_pos_weight` parameter to weight minority class appropriately

**Data split:**
- Training: 60% (Jan 2023 – Aug 2025)
- Validation: 20% (Sep 2025 – Nov 2025)
- Test: 20% (Dec 2025 – May 2026)
- Temporal split (not random) to test model on newer data

---

# Model Training & Performance

## Training Process

```
Step 1: Prepare training data
├─ Query 15M waitlist bookings from IRCTC DB
├─ Filter out: Cancellations on day-of-travel (noise)
├─ Extract features (as listed above)
└─ Normalize numerical features (0–1 scale)

Step 2: Train XGBoost classifier
├─ Hyperparameters:
│  ├─ max_depth: 7 (prevent overfitting)
│  ├─ learning_rate: 0.05 (small steps)
│  ├─ n_estimators: 500 (number of trees)
│  ├─ scale_pos_weight: 2.3 (weight minority class)
│  └─ subsample: 0.8 (use 80% of data per tree)
├─ Cross-validation: 5-fold to measure stability
└─ Training time: ~4–6 hours on 4-CPU machine

Step 3: Hyperparameter tuning
├─ Grid search over learning_rate (0.01–0.1)
├─ Grid search over max_depth (4–10)
└─ Pick best params based on validation accuracy

Step 4: Validate on test set (held-out newer data)
├─ Measure accuracy, precision, recall, ROC-AUC
└─ Confirm generalization to new patterns

Step 5: Deploy to production
├─ Save model to .pkl file (~100 MB)
├─ Wrap in Flask microservice
├─ Load in memory (instant inference)
└─ Ready to serve predictions
```

## Expected Performance

**Model Accuracy (based on industry benchmarks for railway booking data):**

| Metric | Target | How Measured |
|--|--|--|
| **Accuracy** | 81–84% | (Correct predictions / total predictions) |
| **Precision** | 85%+ | (Of predictions saying "confirm", % actually confirmed) |
| **Recall** | 78–82% | (Of actual confirmations, % caught by model) |
| **ROC-AUC** | 0.88–0.92 | (Ranking quality: does model rank high-confidence cases correctly?) |
| **Calibration** | ±5% | (If model says 78% confidence, actually ~73–83% confirm) |

**Why these targets:**
- **Precision > Accuracy:** Crucial to avoid false positives. If model says "will confirm" but doesn't, user gets disappointed. Better to be conservative.
- **Good calibration:** User trusts "78% means 78%." Miscalibrated model (says 78%, only 50% confirm) breaks trust.

**Real-world validation:**
After deploying to 5% of users for 1 month:
- Predictions compared against actual outcomes
- Fine-tune model if drift detected (routes change, booking behavior changes)
- Update model quarterly with 3 months of new data

---

# How Output is Shown to the User

## User Interface: PNR Enquiry Result with Prediction

**When user checks waitlisted PNR:**

```
┌──────────────────────────────────────────┐
│ PNR ENQUIRY RESULT                        │
├──────────────────────────────────────────┤
│                                          │
│ ⏳ YOUR TICKET IS WAITLISTED              │ ← Status icon
│                                          │
│ PNR: 9876543210                          │
│ Train: 12670 Chennai Express             │
│ Route: Chennai → Delhi                   │
│ Date: May 18, 2026                       │
│ Class: Sleeper | Waitlist Position: #8   │
│ Passengers: 2                            │
│                                          │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                          │
│ 🤖 AI CONFIRMATION PREDICTION             │ ← AI section
│                                          │
│ Based on 3+ years of booking patterns    │
│ for this train, route, and date:        │
│                                          │
│ YOUR CHANCES OF CONFIRMATION:            │
│                                          │
│ ████████░░░░░░░░░░░ 78%                 │ ← Visual bar
│                                          │
│ Confidence: HIGH ⭐⭐⭐⭐⭐              │ ← Stars = confidence
│                                          │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                          │
│ WHAT THIS MEANS:                         │ ← Explanation
│                                          │
│ Good news! Your position usually         │
│ gets confirmed. Based on historical      │
│ cancellations, WL #8 on this route       │
│ in May typically confirms 2–3 days      │
│ before departure.                        │
│                                          │
│ ESTIMATED CONFIRMATION DATE:             │ ← Actionable date
│ May 15, 2026 (3 days before departure)   │
│                                          │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                          │
│ RECOMMENDED NEXT STEPS:                  │ ← Guidance
│                                          │
│ 1. ✓ Check back on May 15 morning       │
│    (when confirmation usually happens)   │
│                                          │
│ 2. [ SET REMINDER ]                      │
│    We'll notify you on May 15 at 6 AM   │
│                                          │
│ 3. [ BOOK BACKUP TICKET ] (optional)     │
│    If you want extra safety, book        │
│    another train & cancel later          │
│                                          │
│ 4. [ FIND ALTERNATIVES ]                 │
│    Confirm seats on other trains:        │
│    • 12622 Tamil Nadu (tomorrow) → ✅    │
│    • 12650 Chennai Mail (May 16) → ✅    │
│                                          │
└──────────────────────────────────────────┘

User feels: ✅ Confident, informed, knows what to do
```

## Secondary Display: My Bookings Page

```
MY BOOKINGS

┌──────────────────────────────────────────────────┐
│ Booking #TAT-20260508-1234          Status: WL   │
│ Train: 12670 Chennai Express                     │
│ Route: Chennai → Delhi | Date: May 18, 2026      │
│                                                  │
│ Position: WL #8                                  │
│ 🤖 Chances: 78% confirmation                     │
│    (Updated 2 hours ago)                         │
│                                                  │
│ [ VIEW DETAILS ] [ SET REMINDER ] [ TRACK ]      │
│                                                  │
│ ✅ You have 2 confirmed bookings for this trip  │
│    (Can cancel one after this is confirmed)     │
│                                                  │
└──────────────────────────────────────────────────┘
```

## Notification: Push Alert on Confirmation Day

```
Push Notification (6 AM on May 15):

📱 IRCTC App Notification
┌─────────────────────────────────────┐
│ ✅ YOUR TICKET MIGHT CONFIRM TODAY! │
│                                     │
│ PNR 9876543210 was at WL #8.        │
│ Based on patterns, it usually       │
│ confirms by afternoon.              │
│                                     │
│ [ CHECK NOW ] [ REMIND LATER ]      │
└─────────────────────────────────────┘
```

---

# Fallback & Error Handling

## Scenario 1: Low Confidence Prediction

**When to trigger:** If model confidence < 70%

```
User sees:

⏳ YOUR TICKET IS WAITLISTED

Position: WL #8
Train: Niche route (few bookings historically)

🤖 PREDICTION:

We don't have enough historical data
for this route to make a confident
prediction.

Expected Confirmation: Unknown

WHAT YOU CAN DO:

1. Check back 2 days before departure
   (this is when confirmations usually
    happen across all routes)

2. [ FIND SIMILAR ROUTES ]
   Confirm seats on popular routes with
   historical confirmation data

3. [ CONTACT SUPPORT ]
   For routes with limited data, our
   support team can advise based on
   current cancellation trends
```

**Why this fallback:**
- Never show a prediction you can't defend
- Rather than guess poorly, admit uncertainty
- Guide user to alternative actions
- Preserve user trust in the system

## Scenario 2: Model Prediction Conflict with Actual Outcome

**Example:** Model predicted "78% will confirm" but user's WL #8 didn't confirm

**How we handle:**
1. **Logging:** Log the misprediction (WL position, actual outcome, model prediction)
2. **Retraining:** Monthly review of mispredictions. If pattern emerges (e.g., "model underpredicts in off-peak"), retrain with corrected weights
3. **User communication:** If user's prediction was wrong, send note: "Your WL didn't confirm. We've adjusted our model for similar cases. Sorry!"
4. **Backup options:** Already show alternatives in the UI, so user isn't left stranded

## Scenario 3: Model Inference Fails (API Down)

**When API call to prediction service fails:**

```
⏳ YOUR TICKET IS WAITLISTED

Position: WL #8

⚠️ Prediction service temporarily unavailable

We usually show a prediction here, but
our AI service is updating. Check back
in a few minutes.

In the meantime:
- Historically, WL positions on this
  route have ~60% confirmation rate
- Check back 2–3 days before departure
- [ SET REMINDER ]
- [ FIND ALTERNATIVES ]
```

**Why this fallback:**
- Graceful degradation (not a breaking error)
- Provide aggregate statistics as backup
- Service continues to be useful
- Encourage user action while model retrains

## Scenario 4: New Routes Not in Training Data

**When:** User books WL on brand-new route (launched in past month)

```
⏳ YOUR TICKET IS WAITLISTED

Position: WL #5

🤖 PREDICTION:

This is a new route! We don't yet have
historical booking data to predict
confirmation chances.

What we know:
- Similar routes (similar distance,
  demand) have ~65% confirmation for WL #5
- New routes often have lower confirmation
  initially (fewer cancellations)

[ FIND SIMILAR ROUTES ] [ SET REMINDER ]

Prediction will improve after 1–2 months
of data collection.
```

---

# Technical Implementation

## System Architecture

```
┌─────────────────────────────────────────────────┐
│ IRCTC Backend                                   │
├─────────────────────────────────────────────────┤
│                                                 │
│ [GET /pnr/enquire/{pnr}]                        │ ← User requests PNR
│           │                                     │
│           ↓                                     │
│ [Fetch booking details from DB]                 │
│           │                                     │
│           ├─ If status = WAITLISTED:            │
│           │       │                             │
│           │       ↓                             │
│           │  [Call ML Prediction Service]       │
│           │  (via internal API)                 │
│           │       │                             │
│           │       ↓                             │
│           │  ┌─────────────────────────────┐   │
│           │  │ XGBoost Prediction Service  │   │
│           │  │ (Microservice, Flask)       │   │
│           │  ├─────────────────────────────┤   │
│           │  │ Input: route_id, wl_pos,   │   │
│           │  │        date, class, etc.    │   │
│           │  │                             │   │
│           │  │ XGBoost model (in memory)   │   │
│           │  │ Runs prediction             │   │
│           │  │                             │   │
│           │  │ Output: prob, confidence,   │   │
│           │  │         est_confirm_date    │   │
│           │  └─────────────────────────────┘   │
│           │       ↑                             │
│           │       │ Returns: {                  │
│           │       │   probability: 0.78,       │
│           │       │   confidence: "high",      │
│           │       │   est_date: "2026-05-15"   │
│           │       │ }                          │
│           │       │                             │
│           └────────┘ Merge into response        │
│           │                                     │
│           ↓                                     │
│ Return JSON: {                                  │
│   pnr, status, passenger_details,              │
│   prediction: {                                 │
│     probability: 0.78,                          │
│     confidence: "high",                         │
│     estimated_confirmation_date: "2026-05-15"  │
│   }                                             │
│ }                                               │
│                                                 │
└─────────────────────────────────────────────────┘

Prediction Service Details:
- Type: Containerized microservice (Docker)
- Language: Python (XGBoost, Flask)
- Memory: 200 MB (model file) + overhead
- CPU: 1 core sufficient (predictions <5ms each)
- Response time: 50–200 ms (including network latency)
- Availability: 99.5% uptime SLA (restart on failure)
```

## Deployment Steps

1. **Prepare training data:**
   - Query 15M booking records from data warehouse
   - Feature engineering (1 week)

2. **Train model:**
   - Run XGBoost training pipeline (4–6 hours)
   - Validation on held-out test set
   - Save model file to disk

3. **Package service:**
   ```dockerfile
   FROM python:3.11
   COPY xgboost_model.pkl /app/
   COPY flask_server.py /app/
   RUN pip install xgboost flask
   CMD ["python", "flask_server.py"]
   ```

4. **Deploy:**
   - Docker registry: Push image to IRCTC container registry
   - Kubernetes: Deploy 3 replicas for redundancy
   - Load balancer: Direct `/predict` requests to service
   - Fallback: If service fails, return None (graceful degradation)

5. **Monitor:**
   - Track prediction latency (< 200ms alert)
   - Track inference accuracy monthly (compare predictions vs actual outcomes)
   - Retrain quarterly with 3 months of new data

---

# Success Metrics

| Metric | Baseline | Target | How Measured |
|--|--|--|--|
| **User trust in predictions** | N/A | 85%+ agree "predictions are helpful" | In-app survey: "Was this prediction useful?" (1–5 scale) |
| **Unnecessary rebookings** | 18% of WL users | <5% of WL users | Booking analytics: rebooking_within_1_week / total_wl_bookings |
| **Support tickets about WL status** | 12% of PNR enquiries | <2% | Support categorization |
| **Prediction accuracy** | N/A | 81–84% | Model validation: correct_predictions / total |
| **Feature adoption** | N/A | 60%+ of WL users | Feature usage: users_viewing_prediction / total_wl_enquiries |
| **User confidence (post-booking)** | 2.8/5 | 4.7/5 | Post-journey survey: "How confident were you in this WL status?" |
| **Secondary booking cancellations** | 25% cancel backup tickets | 40% cancel backup | Booking trends: cancellation_rate_for_backup_tickets |

---

# Constraints & Assumptions

1. **Data availability:** IRCTC must provide 3+ years of clean historical booking data. If data quality is poor, model performance degrades.

2. **Route stability:** Model assumes booking patterns are somewhat stable. If new high-speed trains launch or quotas change drastically, model needs retraining.

3. **No causal inference:** Model predicts patterns but doesn't know *why* confirmations happen (it's based on correlations, not causation).

4. **Privacy-compliant:** Model uses aggregate historical data, not individual passenger identities. No PII involved.

5. **Latency budget:** Prediction must complete in <200ms to not slow down PNR enquiry response. Flask service must be co-located with main API servers.

6. **Explainability:** Product team needs to understand model decisions. XGBoost feature importance should be documented.

7. **Regulatory approval:** If model's predictions affect user behavior (e.g., discourage booking), need regulatory signoff from Railway Board.

---

# Why This AI Feature is Better Than Alternatives

| Alternative | Why WL Predictor is Better |
|--|--|
| **Chatbot (NLP-based)** | A chatbot doesn't solve the core problem (user doesn't know if WL will confirm). It's just a conversational interface. Predictor gives actual data-backed answer. |
| **Static rules ("WL < 10 = confirm")** | Rules are too brittle. Off-peak routes: WL #5 rarely confirms. Peak season: WL #20 often confirms. Rules can't adapt. ML model learns these nuances. |
| **Recommending alternates without prediction** | Already done by current system. But users still panic because they don't know their WL chances. Prediction addresses the anxiety directly. |
| **Showing historical aggregate ("60% of WL confirms")** | Useful but not specific. User has WL #8, not average. Model gives tailored prediction: this specific position, this specific route, this specific date. |

---

## Summary

This AI feature is **specific, deployable, and directly solves a documented user pain point.** It doesn't over-promise, has realistic fallbacks, and improves user trust in the IRCTC platform during the most stressful moment—waiting for a waitlisted ticket to confirm.

