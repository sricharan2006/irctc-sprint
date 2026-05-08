# IRCTC Wireframes — Part B

## Overview

This document contains mid-fidelity wireframes for all UI-related feature specs. Each wireframe shows:
- **Mobile layout** (375px width) as primary focus
- **Component labels** and interaction annotations
- **Before/After comparison** showing current state vs proposed solution
- **Error and empty states** for robustness

---

# Wireframe 1: Tatkal Virtual Queue Screen

**Related Spec:** Feature Spec 1 — Tatkal Virtual Queue System
**Problem:** Tatkal booking crashes, no queue visibility, silent failures

## Before State: Current IRCTC Tatkal Screen

```
┌─────────────────────────────────────┐
│  [IRCTC Logo]        [Profile icon] │ ← Header
├─────────────────────────────────────┤
│                                     │
│  TATKAL BOOKING                     │
│                                     │
│  [Search Bar - Disabled]            │ ← Greyed out until 10 AM
│                                     │
│  [Train details pre-filled]         │
│  [Passenger info from previous]     │
│                                     │
│  [Book Now Button - Disabled]       │ ← Not interactive
│                                     │
│  ⏳ ⏳ ⏳ Loading... ⏳ ⏳ ⏳            │ ← Infinite spinner
│                                     │ (actual issue: no feedback)
│                                     │
└─────────────────────────────────────┘

ISSUES SEEN:
× No countdown to 10 AM
× No queue position feedback
× Spinner gives no useful info
× User feels lost and panics
× Leads to refresh spam
```

## After State: Queue Visibility Screen (at 9:55 AM - before opening)

```
┌─────────────────────────────────────┐
│  [IRCTC Logo]        [Profile icon] │
├─────────────────────────────────────┤
│                                     │
│  🕐 TATKAL OPENING IN                │ ← Clear countdown
│  ┌─────────────────────────────────┐│
│  │    0 4  :  2 3  :  1 1           ││ ← Large, bold numbers
│  │    [HH : MM : SS]                ││ ← Updates every second
│  └─────────────────────────────────┘│
│                                     │
│  ✅ Train: 12622 Tamil Nadu Exp      │ ← Pre-confirmation
│     Class: Sleeper | Pax: 2         │
│     Passengers: Confirmed           │
│                                     │
│  [ Change Train ]  [ Edit Passengers] ← Easy modifications
│                                     │
│  ⓘ Pro tip:                         │ ← Setting expectations
│  Your seat preferences are loaded.  │
│  Booking will take <30 sec when     │
│  your turn arrives.                 │
│                                     │
│  [═══════════════════════════════]  │ ← Visual reassurance
│   Ready to join queue               │
│                                     │
└─────────────────────────────────────┘

Tap/Click: "Join Queue" button below when ready
```

## After State: In Queue Screen (at 10:02 AM - actively in queue)

```
┌─────────────────────────────────────┐
│  [IRCTC Logo]        [Profile icon] │
├─────────────────────────────────────┤
│                                     │
│  🎫 YOU'RE IN THE QUEUE               │ ← Reassuring message
│                                     │
│  YOUR POSITION                      │ ← Prominent placement
│  ┌─────────────────────────────────┐│
│  │  #4,281                         ││ ← Position # (human-readable)
│  │  Est. wait: ~9 minutes          ││ ← Time estimate
│  │  ████████░░░░░░░░░░  38% →      ││ ← Progress bar (fills as queue moves)
│  └─────────────────────────────────┘│
│                                     │
│  Your train details:                │ ← Confirmation
│  12622 Tamil Nadu Exp               │
│  Sleeper | 2 passengers             │
│                                     │
│  ℹ️ What's happening:                │ ← User education
│  • 4,300+ people ahead of you       │
│  • Booking opens in your order      │
│  • DO NOT REFRESH - you'll lose     │
│    your position                    │
│  • You'll get a notification when   │
│    your turn arrives                │
│                                     │
│  [Allow Notifications] [Skip]       │
│                                     │
└─────────────────────────────────────┘

Updates in real-time: Position #4,280, Est. 8:47 minutes...
```

## After State: Your Turn Screen (position reached)

```
┌─────────────────────────────────────┐
│  [IRCTC Logo]        [Profile icon] │
├─────────────────────────────────────┤
│                                     │
│  ✅ IT'S YOUR TURN!                   │ ← Celebratory tone
│                                     │
│  ⏱  COMPLETE BOOKING IN               │ ← Countdown for action
│  ┌─────────────────────────────────┐│
│  │  01 : 28                        ││ ← 90-second window (MM:SS)
│  │  (Time remaining)               ││
│  └─────────────────────────────────┘│
│                                     │
│  12622 Tamil Nadu Exp               │
│  Sleeper | 2 passengers             │
│                                     │
│  [ PROCEED TO BERTH SELECTION ]     │ ← Primary action
│                                     │
│  [ABANDON QUEUE]  [EXTEND TIME?*]   │ ← Secondary actions
│                                     │
│  * Call support to extend if needed │
│                                     │
│  ⚠️  If time expires:                │
│  You'll return to queue tail. No    │
│  refund of queue position.          │
│                                     │
└─────────────────────────────────────┘

Interaction notes:
• Countdown timer updates every second
• Visual urgency: color fades to red as time runs out (01:00 red zone)
• Notification sound (optional, respects device settings)
• If user clicks "Proceed" → navigate to normal booking flow (berth selection)
```

## After State: Booking Complete Screen

```
┌─────────────────────────────────────┐
│  [IRCTC Logo]        [Profile icon] │
├─────────────────────────────────────┤
│                                     │
│  🎉 BOOKING CONFIRMED!               │
│                                     │
│  PNR: 1234567890                    │
│  Reference: #TAT-20260508-4281      │
│                                     │
│  12622 Tamil Nadu Exp               │ ← Confirmed details
│  Delhi → Chennai                    │
│  May 15, 2026                       │
│  Sleeper | Berth: Lower 1, Coach A  │
│  Passengers: Raj Kumar, Priya      │
│                                     │
│  ✅ Confirmed                        │ ← Clear status
│  (Railways will issue e-ticket      │
│   within 2 hours)                   │
│                                     │
│  [ DOWNLOAD TICKET ]                │
│  [ SHARE WITH FAMILY ]              │
│  [ VIEW IN MY BOOKINGS ]            │
│                                     │
│  🔔 Check email for confirmation    │
│                                     │
└─────────────────────────────────────┘
```

## Wireframe 2: Search Filters - Persistent State

**Related Spec:** Feature Spec 2 — Persistent Search Filters
**Problem:** Filters reset after refresh, stale data shown

## Before State: Filters Reset After Refresh

```
Screen 1: Initial Search (with filters applied)
┌─────────────────────────────────────┐
│ [IRCTC Logo]        [Menu] [Profile]│
├─────────────────────────────────────┤
│ FROM: Chennai  TO: Delhi    DATE: ▼ │ ← Search bar
│ [Search]                           │
├─────────────────────────────────────┤
│ FILTERS:                            │
│ ☑ Sleeper  ☑ AC2  ☑ AC3            │ ← User selected
│ ☑ General  ☑ Tatkal               │
│ ☑ Available only                   │ ← Filter applied
│ ☑ Morning (0-12)                   │
├─────────────────────────────────────┤
│ RESULTS (2 trains shown):           │
│ 1. 12622 Tamil Nadu Exp - Sleeper   │ ← Filtered correctly
│ 2. 12670 Chennai Express - Sleeper  │
│                                     │
│ [Refresh Page]                      │ ← User accidentally refreshes
└─────────────────────────────────────┘

Screen 2: After Refresh
┌─────────────────────────────────────┐
│ [IRCTC Logo]        [Menu] [Profile]│
├─────────────────────────────────────┤
│ FROM: Chennai  TO: Delhi    DATE: ▼ │
│ [Search]                           │
├─────────────────────────────────────┤
│ FILTERS:                            │
│ ☐ Sleeper  ☑ AC2  ☑ AC3            │ ← Sleeper filter GONE
│ ☐ General  ☑ Tatkal               │ ← General quota GONE
│ ☐ Available only                   │ ← "Available only" unchecked
│ ☐ Morning (0-12)                   │ ← Time filter GONE
├─────────────────────────────────────┤
│ RESULTS (15 trains shown):          │
│ 1. 12622 Tamil Nadu Exp - AC2 WL    │ ← Now showing waitlist!
│    (was filtered out!)              │
│ 2. 12670 Chennai Express - AC3      │
│ 3. [8 more trains with mixed data]  │ ← Stale data mixed in
│                                     │
│ ❌ User frustration: "Where did my │
│    filters go? Now I see WL!"      │
└─────────────────────────────────────┘
```

## After State: Persistent Filters & Smart Caching

```
Screen 1: Search with Filters (persistent)
┌─────────────────────────────────────┐
│ [IRCTC Logo]        [Menu] [Profile]│
├─────────────────────────────────────┤
│ FROM: Chennai  TO: Delhi    DATE: ▼ │
│ [Search]                           │
├─────────────────────────────────────┤
│ ACTIVE FILTERS (pinned):            │ ← Visible at top
│ ✕ Sleeper  ✕ General  ✕ Available   │ ← Easy to remove
│ ✕ Morning (0-12)                    │
│ [Edit Filters ▼]                   │
├─────────────────────────────────────┤
│ RESULTS (2 trains):                 │ ← Accurate matches
│ 1. 12622 Tamil Nadu Exp - Sleeper   │
│    Available: 45 seats              │
│ 2. 12670 Chennai Express - Sleeper  │
│    Available: 12 seats              │
│                                     │
│ ⟳ Last updated: 2 min ago           │ ← Cache metadata
│ [Refresh now]                       │
│                                     │
│ [Refresh Page / Close Tab]          │ ← User refreshes
└─────────────────────────────────────┘

Screen 2: After Refresh (filters PERSIST)
┌─────────────────────────────────────┐
│ [IRCTC Logo]        [Menu] [Profile]│
├─────────────────────────────────────┤
│ FROM: Chennai  TO: Delhi    DATE: ▼ │
│ [Search]                           │
├─────────────────────────────────────┤
│ ACTIVE FILTERS (preserved!):        │ ← Same filters still here
│ ✕ Sleeper  ✕ General  ✕ Available   │
│ ✕ Morning (0-12)                    │
│ [Edit Filters ▼]                   │
├─────────────────────────────────────┤
│ RESULTS (2 trains - freshly cached):│ ← Still accurate
│ 1. 12622 Tamil Nadu Exp - Sleeper   │
│    Available: 43 seats ↓ (was 45)   │ ← Updated availability
│ 2. 12670 Chennai Express - Sleeper  │
│    Available: 12 seats              │
│                                     │
│ ⟳ Last updated: 30 sec ago          │ ← Refreshed! (was 2 min)
│ [Refresh now]                       │
│                                     │
│ ✅ User happiness: "Filters stayed!" │
└─────────────────────────────────────┘

Tech notes:
• Filters stored in localStorage + synced to user account
• Search results cached with 30-min TTL
• Real-time availability updates every 30 sec
• No filter loss on navigation
```

## Wireframe 3: Berth Selection State Persistence

**Related Spec:** Feature Spec 3 — Berth Preference State Persistence
**Problem:** Selected berth resets mid-flow

## Before State: Berth Selection Loss

```
Screen 1: Berth Selection (Step 1b)
┌─────────────────────────────────────┐
│ STEP 1B: SELECT BERTH PREFERENCE     │
│ (12622 Tamil Nadu Exp - Sleeper)     │
├─────────────────────────────────────┤
│ I prefer:                            │
│ ◉ Lower Berth (selected)            │ ← User selects Lower
│ ○ Middle Berth                      │
│ ○ Upper Berth                       │
│ ○ Auto Assign                       │
│                                     │
│ [Next: Passenger Details]           │ ← Click Next
└─────────────────────────────────────┘

Screen 2: Passenger Details (Step 2)
┌─────────────────────────────────────┐
│ STEP 2: PASSENGER DETAILS            │
├─────────────────────────────────────┤
│ Passenger 1:                        │
│ Name: [Raj Kumar]                   │
│ Age: [45]                           │
│ Gender: M                           │
│                                     │
│ Berth Preference: Auto Assign        │ ← Changed to Auto!
│ (Previous selection was: Lower)     │ ← User selected Lower, but now it's Auto
│                                     │
│ ⚠️  User confusion:                  │
│ "I chose Lower but it's Auto now?"  │
│ User has to go back and reselect   │
│                                     │
│ [Previous] [Next]                   │
└─────────────────────────────────────┘
```

## After State: Berth Selection Persisted

```
Screen 1: Berth Selection (Step 1b) - New Flow
┌─────────────────────────────────────┐
│ STEP 1B: SELECT BERTH PREFERENCE     │
│ (12622 Tamil Nadu Exp - Sleeper)     │
├─────────────────────────────────────┤
│ I prefer:                            │
│ ◉ Lower Berth (selected)            │ ← User selects Lower
│ ○ Middle Berth                      │
│ ○ Upper Berth                       │
│ ○ Auto Assign                       │
│                                     │
│ ✅ Reserved: Lower Berth             │ ← System confirms
│    [Confirmation ID: xyZ789]        │ ← Backend reservation made
│    Reserved for 14:59 minutes       │ ← Expiry countdown
│                                     │
│ [Next: Passenger Details]           │ ← Click Next
└─────────────────────────────────────┘

Screen 2: Passenger Details (Step 2) - WITH PERSISTENCE
┌─────────────────────────────────────┐
│ STEP 2: PASSENGER DETAILS            │
│                                     │
│ ✅ YOUR SEAT: Lower Berth, Window    │ ← PERSISTENT BANNER
│    Side, Coach A                    │ ← Visible across all steps
│    [Reserved until 14:58] ⏱         │
│                                     │
├─────────────────────────────────────┤
│ Passenger 1:                        │
│ Name: [Raj Kumar]                   │
│ Age: [45]                           │
│ Gender: M                           │
│                                     │
│ Berth Preference: Lower Berth       │ ← SAME as before!
│ (Your selected preference - locked) │ ← Cannot change without going back
│                                     │
│ ✅ User happiness:                   │
│ "My berth stayed! Great!"           │
│                                     │
│ [Previous] [Next]                   │
└─────────────────────────────────────┘

Screen 3: Payment (Step 3) - Berth Still Visible
┌─────────────────────────────────────┐
│ STEP 3: PAYMENT                      │
│                                     │
│ ✅ YOUR SEAT: Lower Berth, Window    │ ← Still visible!
│    Side, Coach A                    │
│    [Reserved until 14:57] ⏱         │
│                                     │
├─────────────────────────────────────┤
│ Order Summary:                      │
│ Train: 12622 Tamil Nadu Exp         │
│ Route: Delhi → Chennai              │
│ Date: May 15, 2026                  │
│ Berth: Lower 1, Coach A (Window)    │
│ Passengers: Raj Kumar (45M)         │
│ Class: Sleeper                      │
│                                     │
│ Total: ₹500                         │
│                                     │
│ [ PAY WITH NETBANKING ]             │
│ [ PAY WITH CARD ]                   │
│                                     │
└─────────────────────────────────────┘

Key differences:
✓ Persistent banner shows selected berth on every screen
✓ Countdown timer shows reservation validity
✓ Backend validates berth on each step
✓ User cannot accidentally change berth without going back
✓ If berth unavailable: immediate alert with alternatives
```

---

# Wireframe 4: Session Persistence & Auto-Login

**Related Spec:** Feature Spec 4 — Persistent Session Management
**Problem:** Repeated login prompts interrupt flow

## Before State: Unexpected Login Popups

```
User flow with interruptions (current):

Screen 1: Train Search
┌─────────────────────────────────────┐
│ [Search] From Chennai To Delhi ▼    │
│ Browse trains...                    │
│ (10 minutes of browsing)            │
└─────────────────────────────────────┘
                    ↓
[User navigates to Reservation Chart page]
                    ↓
Screen 2: UNEXPECTED LOGIN POPUP!
┌─────────────────────────────────────┐
│ ⚠️  SESSION EXPIRED                   │
│ Please log in again                 │
│                                     │
│ Email: [              ]             │
│ Password: [              ]          │
│ CAPTCHA: [Image]  [Input]           │
│                                     │
│ [LOGIN]                             │
│                                     │
│ ❌ User frustration:                 │
│ "I was just logged in!"             │
│ "I lost my search results!"         │
│ "This is wasting time!"             │
└─────────────────────────────────────┘

Problem summary:
× Session timeout after 30 minutes of browsing
× No advance warning
× Forces user to re-login mid-flow
× Lost navigation state
× Extra time cost during critical booking period
```

## After State: Silent Token Refresh (no interruption)

```
Behind-the-scenes session management (now):

Timeline:
User logs in at 9:45 AM
├─ Access Token issued (valid for 4 hours)
├─ Refresh Token issued (valid for 30 days)
└─ Session stored in Redis

User browses trains for 50 minutes
├─ 10:30 AM: System checks token
├─ Token expires at 1:45 PM - no rush
├─ Silent refresh at 1:30 PM (before expiry)
├─ (User doesn't notice anything)
└─ New access token issued

User continues browsing for another 2 hours
├─ Tokens silently refresh every 4 hours
├─ Session remains valid
└─ No interruption

User is away from phone for 30 minutes
├─ Tokens still valid when they return
├─ No login prompts
├─ Session continues seamlessly

---

Screen: Train Search (no interruption)
┌─────────────────────────────────────┐
│ [Search] From Chennai To Delhi ▼    │
│ Browse trains...                    │
│ (50 minutes of browsing)            │
│                                     │
│ 🔄 Silently refreshing session...   │ ← Transparent to user
│    (no UI change)                   │
└─────────────────────────────────────┘
                    ↓
[User navigates to Reservation Chart page]
                    ↓
Screen: Reservation Chart (no login!)
┌─────────────────────────────────────┐
│ RESERVATION CHART LOADS NORMALLY     │
│ (No login prompt ever shown)         │
│ User sees chart and berth data      │
│                                     │
│ ✅ Session refreshed silently       │
│ ✅ Search results preserved         │
│ ✅ User flow uninterrupted          │
└─────────────────────────────────────┘

User happiness: ✅ Seamless experience
```

## Error State: When Session DOES Fail (graceful)

```
Scenario: User is offline for 20+ minutes, then comes back

Screen: Offline Detection
┌─────────────────────────────────────┐
│ [Previous page state preserved]      │
│                                     │
│ ⚠️  Your session may have expired     │ ← Soft notification
│ (Minimal intrusion)                 │
│                                     │
│ [Continue Browsing]  [Log In Again] │
│                                     │
│ If user clicks "Continue Browsing": │
│ → Try to auto-refresh token         │
│ → If successful: continue (invisible)
│ → If failed: show minimal login form
│                                     │
│ If user clicks "Log In Again":      │
│ → Show quick login with email       │
│    pre-filled                       │
│ → No CAPTCHA if device trusted      │
└─────────────────────────────────────┘

Key differences from old experience:
✓ Session lasts 4 hours (vs 30 min)
✓ Automatic token refresh before expiry
✓ Login only required after full session end (24 hours inactive)
✓ Graceful error state when refresh fails
✓ Device is "remembered" to reduce friction on re-login
```

---

# Wireframe 5: PNR Enquiry - Error Clarity

**Related Spec:** Feature Spec 5 — PNR Error Clarity
**Problem:** Cryptic error messages ("FLUSHED PNR", etc.)

## Before State: Confusing PNR Messages

```
Screen: PNR Enquiry Result (confusing)
┌─────────────────────────────────────┐
│ PNR ENQUIRY RESULT                   │
├─────────────────────────────────────┤
│ PNR: 1234567890                     │
│ Status: FLUSHED PNR                 │ ← What does this mean?
│                                     │
│ ❓ User confusion:                   │
│ "FLUSHED? Does my ticket exist?"    │
│ "Is it good or bad?"                │
│ "Why is it flushed?"                │
│ "What do I do next?"                │
│                                     │
│ [OK]                                │
│                                     │
│ User's next action:                 │
│ 1. Google "FLUSHED PNR meaning"     │
│ 2. Call IRCTC support (wait 30 min) │
│ 3. Panic: "Did I lose my ticket?"   │
│ 4. Rebook backup train (wasted $)   │
│                                     │
│ Cost of confusion: Extra booking    │
│ + wasted time + support ticket      │
└─────────────────────────────────────┘

Another error example:
┌─────────────────────────────────────┐
│ PNR ENQUIRY RESULT                   │
├─────────────────────────────────────┤
│ PNR: 9876543210                     │
│ Status: PNR NOT YET GENERATED       │ ← Technical jargon
│                                     │
│ ❓ Similar confusion...              │
│ "Not generated? But I have a PNR #" │
│ "When will it be generated?"        │
│ "Is there a problem with my booking?"
│                                     │
└─────────────────────────────────────┘
```

## After State: Clear, Actionable PNR Messages

```
Screen: PNR Enquiry Result (clear)
┌─────────────────────────────────────┐
│ PNR ENQUIRY RESULT                   │
├─────────────────────────────────────┤
│                                     │
│ ✅ YOUR TICKET IS CONFIRMED          │ ← Clear status
│                                     │
│ PNR: 1234567890                     │
│ Train: 12622 Tamil Nadu Exp         │
│ Route: Delhi → Chennai              │
│ Date: May 15, 2026                  │
│ Class: Sleeper | Berth: Lower 1     │
│ Passenger: Raj Kumar                │
│                                     │
│ Status: Confirmed by Indian Railways │ ← Human language
│ (Your PNR was processed by our      │ ← Explanation
│  system and confirmed on railways)  │
│                                     │
│ NEXT STEPS:                         │ ← Actionable
│ ✓ Download e-ticket from email      │
│ ✓ Check in 2 hours before departure │
│ ✓ Share PNR with co-travelers       │
│                                     │
│ [ DOWNLOAD TICKET ] [ SHARE ]       │
│                                     │
│ ✅ User clarity: "Ticket confirmed!" │
└─────────────────────────────────────┘

---

Screen: Waitlist Enquiry (with prediction)
┌─────────────────────────────────────┐
│ PNR ENQUIRY RESULT                   │
├─────────────────────────────────────┤
│                                     │
│ ⏳ YOUR TICKET IS WAITLISTED          │ ← Status icon + label
│                                     │
│ PNR: 9876543210                     │
│ Train: 12670 Chennai Express        │
│ Route: Chennai → Delhi              │
│ Date: May 18, 2026                  │
│ Class: Sleeper | Position: WL #8    │
│ Passengers: 2                       │
│                                     │
│ Status: Waiting List #8             │ ← Clear position
│ (Your booking is in queue to be     │
│  confirmed as cancellations happen) │
│                                     │
│ 🤖 CONFIRMATION PREDICTION:         │ ← ML-powered insight
│ Based on historical booking patterns │
│ for this train & route, your        │
│ position #8 usually confirms        │
│ 2-3 days before departure.          │
│                                     │
│ Confidence: 78% ⭐⭐⭐⭐             │
│                                     │
│ WHAT THIS MEANS:                    │
│ Good news: High chance of           │
│ confirmation                        │
│                                     │
│ CHECK AGAIN:                        │ ← Smart reminder
│ May 15 (3 days before departure)    │
│ Status usually updates in your favor │
│                                     │
│ [ SET REMINDER ]  [ CHECK AGAIN ]   │
│                                     │
│ If not confirmed by May 17:         │ ← Backup plan
│ - Check 5 alternatives trains       │
│ - Consider booking backup ticket    │
│ - We'll help you switch             │
│                                     │
│ ✅ User clarity: "I have 78% chance" │
└─────────────────────────────────────┘

---

Screen: Error - PNR Not Found (helpful)
┌─────────────────────────────────────┐
│ PNR ENQUIRY RESULT                   │
├─────────────────────────────────────┤
│                                     │
│ ❓ PNR NOT FOUND                     │
│                                     │
│ We couldn't find PNR: 1234567890    │ ← Acknowledge
│                                     │
│ POSSIBLE REASONS:                   │ ← Helpful guidance
│ 1. PNR number entered is incorrect  │
│    (Double-check spelling)          │
│ 2. PNR was just booked              │
│    (Railways take 15-30 min to      │
│    sync. Try again in 10 min.)      │
│ 3. Booking was cancelled            │
│    (Check your email for details)   │
│                                     │
│ WHAT TO TRY:                        │ ← Next steps
│ [ SEARCH BY BOOKING REFERENCE ]     │
│ [ SEARCH BY PHONE NUMBER + DATE ]   │
│ [ CONTACT SUPPORT ]                 │
│                                     │
│ TIP:                                │ ← Preventive
│ Copy your PNR when you book.        │
│ Save it to email/notes to avoid     │
│ typing errors.                      │
│                                     │
│ ✅ User clarity: "Here's what to try" │
└─────────────────────────────────────┘
```

---

# Wireframe 6: Reservation Chart - Visual Redesign

**Related Spec:** Feature Spec 6 — Reservation Chart Redesign
**Problem:** Dense table layout difficult to scan

## Before State: Current Dense Table

```
RESERVATION CHART - Train 12622, Sleeper, May 15, 2026

┌─────────────────────────────────────────────────────────────┐
│ Coach │ Berth │ Type    │ Status │ Price │ Quota │ Side │ Res │
├─────────────────────────────────────────────────────────────┤
│ A1    │ 1     │ Lower   │ BOOKED │ 500   │ GN    │ L    │ PNR │
│ A1    │ 2     │ Middle  │ BOOKED │ 480   │ GN    │ L    │ PNR │
│ A1    │ 3     │ Upper   │ AVAIL  │ 480   │ GN    │ L    │ -   │
│ A1    │ 4     │ Side-L  │ AVAIL  │ 520   │ GN    │ L    │ -   │
│ A1    │ 5     │ Side-U  │ AVAIL  │ 520   │ GN    │ L    │ -   │
│ A1    │ 6     │ Lower   │ BOOKED │ 500   │ GN    │ R    │ PNR │
│ A1    │ 7     │ Middle  │ AVAIL  │ 480   │ GN    │ R    │ -   │
│ A1    │ 8     │ Upper   │ AVAIL  │ 480   │ GN    │ R    │ -   │
│ A2    │ 1     │ Lower   │ AVAIL  │ 500   │ GN    │ L    │ -   │
│ A2    │ 2     │ Middle  │ BOOKED │ 480   │ GN    │ L    │ PNR │
│ [... 20+ more rows ...]                              │
└─────────────────────────────────────────────────────────────┘

PROBLEMS with this view:
❌ Dense rows - hard to scan
❌ No visual hierarchy
❌ Colors all the same (black text)
❌ "BOOKED" vs "AVAIL" - not immediately obvious
❌ No berth structure visualization
❌ Mobile view is impossible to read
❌ Side (L/R) and Type not intuitive
❌ User must read every row carefully
❌ Takes 2-3 minutes to find available berth

User experience: "This is so confusing..."
```

## After State: Visual Berth Grid Layout

```
┌─────────────────────────────────────────────────────────────┐
│ [IRCTC Logo]            RESERVATION CHART     [Filter ▼]   │
│ Train 12622 Tamil Nadu Exp | Sleeper | May 15, 2026        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ COACH SELECTION (Carousel):                                │
│ ◄ [ A1 ]  [ A2 ]  [ A3 ]  [ A4 ]  [ A5 ] ►               │
│      ↑ Currently viewing                                   │
│                                                             │
│ Coach A1: 5/8 seats available | 3 booked                  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  BERTH GRID (Visual Layout):                               │
│                                                             │
│  LEFT SIDE          │        RIGHT SIDE                    │
│  ─────────────────────────────────────────                 │
│  ┌─────┐  ┌─────┐   │  ┌─────┐  ┌─────┐                  │
│  │  1  │  │  4  │   │  │  6  │  │  9  │                  │
│  │ ■   │  │ ✓   │   │  │ ■   │  │ ✓   │                  │
│  │ Low │  │ SidL│   │  │ Low │  │ Mid │                  │
│  │ ₹500│  │₹520 │   │  │ ₹500│  │₹480 │                  │
│  └─────┘  └─────┘   │  └─────┘  └─────┘                  │
│  ┌─────┐  ┌─────┐   │  ┌─────┐  ┌─────┐                  │
│  │  2  │  │  5  │   │  │  7  │  │ 10  │                  │
│  │ ■   │  │ ✓   │   │  │ ✓   │  │ ✓   │                  │
│  │ Mid │  │ SidU│   │  │ Mid │  │ Upp │                  │
│  │ ₹480│  │₹520 │   │  │ ₹480│  │₹480 │                  │
│  └─────┘  └─────┘   │  └─────┘  └─────┘                  │
│  ┌─────┐  ┌─────┐   │  ┌─────┐  ┌─────┐                  │
│  │  3  │  │  *  │   │  │  8  │  │ 11  │                  │
│  │ ✓   │  │  X  │   │  │ ✓   │  │  -  │                  │
│  │ Upp │  │Block│   │  │ Upp │  │ RAC │                  │
│  │ ₹480│  │     │   │  │ ₹480│  │     │                  │
│  └─────┘  └─────┘   │  └─────┘  └─────┘                  │
│                                                             │
│  Legend:                                                   │
│  ✓ Available (green) │ ■ Booked (red) │ X Blocked │ RAC   │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SMART FILTERS:                                            │
│  [Show All] [Available Only] [Lower Berths] [Window]      │
│                                                             │
│  SELECTED BERTH DETAILS:                                  │
│  (Click any berth to see details)                         │
│  ┌─────────────────────────────────────────────┐          │
│  │ BERTH #4 - Side Lower, Window Side          │          │
│  │ Price: ₹520                                 │          │
│  │ Coach: A1 (Left window)                     │          │
│  │ Preferences: Window seat, accessible        │          │
│  │ This berth is good for elderly passengers   │          │
│  │                                             │          │
│  │ [ SELECT THIS BERTH ] [ EXPLORE OTHERS ]   │          │
│  └─────────────────────────────────────────────┘          │
│                                                             │
└─────────────────────────────────────────────────────────────┘

ADVANTAGES of this view:
✅ VISUAL - immediate color recognition (green=available)
✅ STRUCTURED - berth grid shows actual coach layout
✅ SCANNABLE - user finds available berth in 10 seconds
✅ MOBILE-FRIENDLY - large berth tiles (touch targets)
✅ INFORMATIVE - shows price, type, side in one view
✅ INTERACTIVE - click berth to see details
✅ EDUCATIONAL - "Good for elderly" micro-hints
✅ HIERARCHY - key info (berth #, type, price) prominent

User experience: ✅ "Found available berth in 15 seconds!"

---

Mobile View (375px):

┌──────────────────────────────────────┐
│ RESERVATION CHART              [↻] │
│ 12622 | Sleeper | May 15           │
├──────────────────────────────────────┤
│                                      │
│ COACH: ◄ [ A1 ] ► (5/8 available)   │
│                                      │
│ LEFT SIDE:      RIGHT SIDE:         │
│ ┌───┐ ┌───┐    ┌───┐ ┌───┐        │
│ │ 1 │ │ 4 │    │ 6 │ │ 9 │        │
│ │■  │ │✓  │    │■  │ │✓  │        │
│ │L  │ │SL │    │L  │ │M  │        │
│ └───┘ └───┘    └───┘ └───┘        │
│ ┌───┐ ┌───┐    ┌───┐ ┌───┐        │
│ │ 2 │ │ 5 │    │ 7 │ │10 │        │
│ │■  │ │✓  │    │✓  │ │✓  │        │
│ │M  │ │SU │    │M  │ │U  │        │
│ └───┘ └───┘    └───┘ └───┘        │
│ ┌───┐ ┌───┐    ┌───┐ ┌───┐        │
│ │ 3 │ │ * │    │ 8 │ │11 │        │
│ │✓  │ │X  │    │✓  │ │-  │        │
│ │U  │ │BL │    │U  │ │RAC│        │
│ └───┘ └───┘    └───┘ └───┘        │
│                                      │
│ Filter: [Available] [Lower] [Window]│
│                                      │
│ Tap berth #4 (Green) for details    │
│                                      │
├──────────────────────────────────────┤
│ SELECTED: Berth #4                   │
│ Side Lower, Window                   │
│ Price: ₹520                          │
│ Good for: Elderly, families          │
│ [ SELECT ]                           │
│                                      │
└──────────────────────────────────────┘

Mobile improvements:
✅ Readable on 2G connection (minimal data)
✅ Large touch targets (40x40px berths)
✅ Coach carousel (one at a time)
✅ Color + icons (accessible to color-blind)
✅ Scan time: 8 seconds to find berth
```

---

## Summary: Wireframe Coverage

| Problem | Wireframe | UI Components | Key Interactions |
|---------|-----------|---------------|-----------------|
| Problem 1: Tatkal Queue | 1 | Queue counter, countdown, progress bar | Join queue, view position, book when called |
| Problem 2: Search Filters | 2 | Filter chips, result list, cache badge | Apply filters, persist across refresh |
| Problem 3: Berth Selection | 3 | Berth selector, preference banner | Select berth, persist across steps |
| Problem 4: Session | 4 | Silent refresh, login modal | Auto-refresh tokens, graceful fallback |
| Problem 5: PNR Errors | 5 | Status cards, prediction, actions | View clarity, set reminders, find alternatives |
| Problem 6: Chart Layout | 6 | Berth grid, carousel, detail panel | Select berth, filter, navigate coaches |

