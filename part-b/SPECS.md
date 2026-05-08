# IRCTC Feature Specifications — Part B

## Overview

This document contains detailed feature specifications for all 6 problems identified in Part A. Each specification bridges discovery and delivery by answering four critical questions: **What problem are we solving?** **How does the user experience this solution?** **What changes in the system?** **How do we know it worked?**

---

# Feature Spec 1: Tatkal Virtual Queue System

**Traceability:** [Part A — Problem 1: Tatkal Booking Crashes at 10:00 AM](../part-a/PROBLEMS.md#problem-1-tatkal-booking-crashes-at-1000-am-given)

## Problem Statement

During Tatkal opening at 10:00 AM, the IRCTC platform receives 20–40 lakh concurrent requests within seconds. The system lacks queue management and provides no feedback to users, causing HTTP 502 errors, session timeouts, CAPTCHA resets, and payment uncertainty. Users panic-refresh, further overloading servers. The platform cannot distinguish between first-time requesters and automatic refreshes, leading to a cascading failure cascade that blocks legitimate booking attempts.

## Proposed Solution

Implement a client-side virtual queue system that manages incoming booking requests with transparent progress tracking. Users are assigned a position in the queue immediately upon clicking "Book Now." The UI displays:
- Live countdown to their booking window
- Current queue position (e.g., "#4,281")
- Estimated wait time based on historical booking speed
- Progress bar showing queue progression
- When their position is reached, they have 90 seconds to complete the booking before their slot expires and they return to the queue tail

This system prevents request storms hitting the backend simultaneously and gives users realistic expectations instead of silent freezes.

## Technical Implementation Plan

### System Components Affected
1. Frontend: New queue management module and live counter UI component
2. Backend API: New queue service (state management + position tracking)
3. Database: New `tatkal_queue` table to store queue state
4. Real-time layer: WebSocket or Server-Sent Events (SSE) for live position updates
5. Payment gateway: Modified booking timeout logic (90-second booking window)

### New Data Requirements
**`tatkal_queue` table:**
- `queue_id` (UUID, primary key)
- `user_id` (FK to users table)
- `position` (integer, queue ordering)
- `entered_at` (timestamp)
- `status` (queued/processing/completed/expired)
- `assigned_slot_time` (datetime when user's turn arrives)
- `session_token` (unique identifier for this queue session)

**`booking_attempts` table (audit log):**
- `attempt_id` (UUID)
- `queue_id` (FK)
- `train_id`, `class`, `passengers` (booking parameters)
- `status` (success/expired/failed)
- `completed_at` (timestamp)

### API Changes

**1. POST `/tatkal/queue/join`**
- Request: `{ train_id, date, class_type, passenger_count, user_id }`
- Response: `{ queue_id, position, estimated_wait_seconds, session_token }`
- Purpose: Add user to queue upon "Book Now" click

**2. GET `/tatkal/queue/status/{queue_id}`**
- Response: `{ position, estimated_wait, your_turn_at, status }`
- Purpose: Client polls every 2 seconds to update UI (or uses WebSocket for real-time push)

**3. POST `/tatkal/booking/process`**
- Request: `{ queue_id, session_token, train_id, passengers, payment_token }`
- Response: `{ status, pnr, confirmation_details OR error_reason }`
- Purpose: Process booking only when user's queue position is active
- Constraint: Request rejected if `session_token` is expired or position not reached

**4. POST `/tatkal/queue/abandon`**
- Request: `{ queue_id }`
- Response: `{ status: "abandoned" }`
- Purpose: User explicitly leaves queue to free their slot

### Frontend State Changes

**New Queue Manager Component:**
```
QueueManager (state)
├── queue_id
├── position (number)
├── estimated_wait (seconds)
├── status (queued|your_turn|processing|completed|expired)
├── session_token
└── countdown_interval (live ticker)

New UI Components:
├── QueueStatusScreen (shows position, progress, wait time)
├── BookingWindow (90-second timer when position reached)
└── QueueExpiredModal (if session times out)
```

**Real-time Updates:**
- Use WebSocket (Socket.io) or Server-Sent Events (SSE) instead of polling
- SSE preferred for simplicity: server pushes position updates every 2 seconds
- Fallback to polling if SSE unavailable (for older browsers)
- Push notification when user's turn arrives

### Third-party Services
- **Firebase Cloud Messaging (FCM)** or **OneSignal**: Push notifications to app/browser when position reaches
- **Redis** (optional, for very fast queue state cache if DB becomes bottleneck)

### Edge Cases and Constraints

1. **Network dropout during booking window**: Store queue state in browser localStorage. On reconnect, validate `session_token` is still active and resume booking.

2. **User closes browser before booking starts**: Queue position is released after 5 minutes of inactivity. User can rejoin from scratch.

3. **90-second booking window expires**: User is moved to tail of queue (lowest priority slot) or exit with option to rejoin.

4. **Payment gateway timeout**: If payment takes >90 seconds, flag for manual review but don't immediately reject—check if payment eventually went through.

5. **IRCTC railway backend API unavailable**: Show "Backend temporarily unavailable. Your queue position is preserved. We'll retry when service is back."

6. **Extreme scale (50+ lakh users)**: Implement queue sharding by route/date to split load. Queue size per shard: 5-10k users max.

7. **Seat inventory exhausted while user is booking**: Show top 3 alternative classes/trains on same route in real-time. Allow quick pivot without losing queue position credit.

## Success Metrics

| Metric | Current State | Target | Measurement Method |
|--------|---------------|--------|-------------------|
| Tatkal booking completion rate (peak hours) | 40% | 75%+ | Dashboard: (completed bookings / queue joins) |
| Avg. booking time (from queue entry to confirmation) | 180 sec | 120 sec | Queue logs: booking_time = completed_at - assigned_at |
| HTTP 502 errors during Tatkal | 8–12% of peak traffic | <1% | Error logs: count 502s / total requests during 10:00-10:05 AM |
| User panic-refreshes per booking attempt | 15–20 | <2 | Browser logs: detect manual refreshes while queue visible |
| Session timeout rate | 25% | <5% | Auth logs: session_timeout_count / total sessions |
| Queue wait time accuracy | N/A | ±15% | Compare predicted_wait vs actual_wait for 1000+ bookings |
| Payment failure rate (Tatkal) | 12% | <3% | Payment gateway logs: failures during valid booking window |

## Constraints and Assumptions

1. **Government approval**: IRCTC board must approve queue logic change (political sensitivity of "fairness")
2. **Railway backend capacity**: Indian Railways backend APIs must support booking throughput of 5k bookings/min (currently 500/min)
3. **Session security**: Queue token must be cryptographically secure and non-transferable
4. **Mobile compatibility**: Queue UI must work on 2G connections (minimal animations, text-heavy)
5. **Compliance**: Queue ordering must comply with Indian Railways anti-fraud policies (no queue-jumping, no selling queue positions)

---

# Feature Spec 2: Persistent Search Filters with Smart Caching

**Traceability:** [Part A — Problem 2: Search Filters Do Not Work Reliably](../part-a/PROBLEMS.md#problem-2-search-filters-do-not-work-reliably-given)

## Problem Statement

Train search filters reset after page refresh, forcing users to re-select class, quota, departure time, and availability constraints repeatedly. Stale availability data is displayed even after filters are applied, showing waitlisted trains in "Available seats only" results. This frustrates users during high-traffic periods and consumes extra booking time, particularly for first-time users and senior citizens unfamiliar with the interface.

## Proposed Solution

Implement persistent filter state management with client-side caching and smart backend synchronization. Filters are preserved across page refreshes, back navigation, and session returns. Search results are cached and automatically refreshed every 30 seconds with updated availability data. Users see a "Last updated: 2 minutes ago" indicator and can manually trigger refresh. Filter state is also synced to user's account (not just browser localStorage) so they see the same filters across devices.

## Technical Implementation Plan

### System Components Affected
1. Frontend: New state management (Redux/Context API) for filter persistence
2. Frontend: Client-side search cache with TTL (time-to-live)
3. Backend API: Modified search endpoint to include cache metadata
4. Database: New `user_search_preferences` table for filter sync across devices
5. Cache layer: Redis for backend search result caching

### New Data Requirements

**`user_search_preferences` table:**
- `preference_id` (UUID)
- `user_id` (FK to users)
- `source_station` (string)
- `dest_station` (string)
- `travel_date` (date)
- `class_filters` (JSON: ["Sleeper", "AC2"])
- `quota_filters` (JSON: ["General", "Tatkal"])
- `availability_filter` (enum: "all" / "available_only" / "waitlist_only")
- `time_range` (JSON: {start_hour: 0, end_hour: 24})
- `sort_by` (enum: "departure" / "arrival" / "duration" / "price")
- `created_at`, `updated_at`, `last_accessed_at`

**`search_results_cache` (Redis key-value):**
- Key: `search:{source}:{dest}:{date}:{filter_hash}`
- Value: `{ trains: [...], cached_at, ttl: 30_minutes }`

### API Changes

**1. POST `/search/trains` (modified)**
- Request: 
  ```json
  {
    "source": "Chennai",
    "destination": "Delhi",
    "date": "2026-05-15",
    "filters": {
      "class": ["Sleeper", "AC2"],
      "quota": ["General"],
      "availability": "available_only",
      "time_range": {"start": 0, "end": 24}
    },
    "sort_by": "departure"
  }
  ```
- Response:
  ```json
  {
    "trains": [...],
    "cached": true,
    "cached_at": "2026-05-08T14:32:10Z",
    "ttl_seconds": 1800,
    "next_refresh_at": "2026-05-08T15:02:10Z",
    "filter_state": { /* echo back the applied filters */ }
  }
  ```

**2. GET `/search/preferences` (new)**
- Response: `{ source, dest, date, filters, last_accessed_at }`
- Purpose: Retrieve user's saved filter preferences

**3. POST `/search/preferences/save` (new)**
- Request: Filter object to save as user's default
- Response: `{ saved: true, preference_id }`
- Purpose: Save current filter set as user's default for future searches

**4. GET `/search/trains/{train_id}/availability` (new)**
- Response: Real-time availability check (not cached)
- Purpose: When user clicks on a specific train, fetch fresh availability to avoid stale data

### Frontend State Changes

**New Filter Manager (Redux):**
```javascript
filterState = {
  source: "Chennai",
  destination: "Delhi",
  date: "2026-05-15",
  filters: {
    class: ["Sleeper", "AC2"],
    quota: ["General"],
    availability: "available_only",
    timeRange: [0, 24]
  },
  sortBy: "departure",
  
  // Cache metadata
  cached: true,
  cachedAt: timestamp,
  nextRefreshAt: timestamp,
  cacheStale: boolean
}
```

**Client-side cache logic:**
- Store filter state in Redux + browser localStorage
- Store search results with expiry time
- On route change / page navigation, restore from localStorage
- Every 30 seconds or on manual refresh, check cache TTL and fetch fresh results

**UI Changes:**
- Add "Last updated X minutes ago" badge on search results
- Add "Refresh results" button to trigger immediate update
- Show loading state when filters change vs when cache is being refreshed
- Persist filter selections in a collapsible "Active Filters" panel

### Third-party Services
- None (Redis is internal infrastructure)

### Edge Cases and Constraints

1. **Filter becomes invalid (e.g., no trains for that route on that date)**: Show "No trains found matching your filters" and suggest removing filters one by one.

2. **User changes device mid-search**: Load last saved preferences from `user_search_preferences` table on login.

3. **Search cache grows too large**: Implement cache eviction policy: LRU (Least Recently Used) for Redis. Purge individual cache entries older than 1 hour.

4. **User applies 10+ filter combinations in 1 minute**: Throttle API calls to max 1 per 5 seconds to prevent accidental DoS.

5. **Mobile user with slow connection**: Reduce cache check interval to 60 seconds instead of 30 (less data usage).

6. **Filter state conflicts during sync across devices**: Latest update wins (timestamp-based conflict resolution). Alert user: "Filters updated on another device."

7. **Availability changes between cache and actual booking**: When user clicks "Book Now," re-fetch availability immediately. If seat no longer exists, suggest alternatives.

## Success Metrics

| Metric | Current State | Target | Measurement |
|--------|---------------|--------|------------|
| Filter persistence across page refresh | 0% (resets) | 98%+ | Test: apply filter → refresh → check filter state |
| Stale availability data incidents | 8% of bookings | <1% | Booking failure logs: seats unavailable after filtering |
| Avg. time to find train (with filters) | 120 sec | 45 sec | UX testing: start time → train selection time |
| Filter state accuracy across devices | N/A | 95%+ | Sync test: set filters on mobile → check on desktop |
| Cache hit rate | N/A | 60%+ | Backend logs: cache_hits / total_requests |
| User satisfaction with filters | 3.2/5 | 4.5/5 | Post-booking survey (NPS metric) |

---

# Feature Spec 3: Berth Preference State Persistence

**Traceability:** [Part A — Problem 3: Seat Selection Resets Randomly](../part-a/PROBLEMS.md#problem-3-seat-selection-resets-randomly-given)

## Problem Statement

Selected berth preferences (e.g., lower berth, window seat, aisle) disappear during the multi-step booking flow. Users select a specific berth, proceed to passenger details, and find the system has reset their preference to "Auto" or assigned a different berth entirely. This is especially frustrating for elderly passengers, families with children, and users with mobility constraints. The issue occurs more frequently on mobile devices due to state loss during network transitions.

## Proposed Solution

Implement immutable berth selection state management that persists across the entire booking flow using both client-side storage and backend validation. When a user selects a berth, it is:
1. Stored in browser session storage immediately
2. Sent to backend API to reserve the berth preference
3. Validated on each subsequent step
4. Displayed with visual confirmation ("Your selected: Lower 1, Window Side")
5. If berth becomes unavailable, user is immediately notified with alternative suggestions

## Technical Implementation Plan

### System Components Affected
1. Frontend: Session-level state manager (React Context or Redux) for berth selection
2. Frontend: Client-side validation before proceeding to next step
3. Backend API: New berth reservation endpoint and validation logic
4. Database: New `berth_selections` table to track selected preferences
5. Seat inventory sync: Real-time updates to detect berth unavailability

### New Data Requirements

**`berth_selections` table:**
- `selection_id` (UUID, primary key)
- `user_id` (FK)
- `train_id` (FK)
- `class_type` (enum)
- `preferred_berth_type` (enum: "Lower" / "Middle" / "Upper" / "Side-Lower" / "Side-Upper" / "Window" / "Aisle" / "Auto")
- `selected_berth_id` (specific berth number if user chose exact berth)
- `coach_preference` (if user specified coach section)
- `booking_session_id` (unique ID for this booking attempt)
- `created_at`, `expires_at` (TTL of 15 minutes)
- `status` (active/confirmed/expired)

### API Changes

**1. POST `/berth-selection/reserve` (new)**
- Request:
  ```json
  {
    "train_id": "12622",
    "class_type": "Sleeper",
    "preferred_type": "Lower",
    "specific_berth_id": null,
    "booking_session_id": "abc123"
  }
  ```
- Response:
  ```json
  {
    "selection_id": "xyz789",
    "status": "reserved",
    "preferred_type": "Lower",
    "confirmation_token": "token123"
  }
  ```

**2. GET `/berth-selection/status/{selection_id}` (new)**
- Response: `{ status, preferred_type, confirmation_token, expires_at }`
- Purpose: Validate selection still active before each booking step

**3. GET `/berth-selection/alternatives` (new)**
- Request: If preferred berth no longer available, fetch alternatives
- Response: `{ alternatives: [{ berth_id, type, availability_percentage }] }`

**4. POST `/booking/confirm` (modified)**
- Now requires: `berth_selection_id` and `confirmation_token` in request
- Validation: Confirm berth preference is still valid before finalizing booking

### Frontend State Changes

**Berth Selection Manager (React Context):**
```javascript
berthState = {
  bookingSessionId: "abc123",
  selectionId: "xyz789",
  trainId: "12622",
  classType: "Sleeper",
  preferredType: "Lower",  // User's preference
  specificBerthId: 23,     // Specific berth if chosen
  confirmationToken: "token123",
  
  reservationExpiresAt: timestamp,
  status: "reserved",  // reserved / confirmed / expired
  
  // Fallback data
  alternativePreferences: []
}
```

**Booking Flow Enhancement:**
- Step 1 (Train selection) → New step 1b (Berth selection) → Step 2 (Passenger details) → Step 3 (Payment)
- On leaving Step 1b, call `/berth-selection/reserve` and store `selectionId`
- Before Step 2, call `/berth-selection/status/{selectionId}` to validate
- If validation fails, show "Your berth preference expired. Select again?" with alternatives
- On Step 3 (Payment), include `selectionId` in booking confirmation

**UI Enhancements:**
- Show persistent "Your seat: Lower Berth, Window Side, Coach A" banner across all steps
- Add progress indicator showing which seats are reserved vs available
- On mobile, keep berth selection visible in a sticky header
- Show countdown timer: "Berth reserved for 14:23 more minutes"

### Third-party Services
- None

### Edge Cases and Constraints

1. **Berth becomes unavailable after user selects it**: Immediately notify user with alert + alternatives. Allow 1-click switch without losing progress.

2. **User leaves booking flow for 15 minutes**: Berth reservation expires. On return, show "Your reservation expired. Try again?" with fresh availability.

3. **Network loss during berth selection**: Store preference in localStorage. On reconnect, resync with backend to confirm reservation.

4. **User is on mobile switching between apps**: Session storage persists across app return within 5 minutes. After 5 minutes, require re-selection.

5. **Berth class changes (e.g., Sleeper now full, only AC available)**: If user wants to pivot to new class, warn them: "Your berth preference doesn't exist in AC 2 tier. Choose new preference?"

6. **Multiple bookings in same session**: Each booking has separate `booking_session_id` to prevent cross-contamination of preferences.

7. **Co-traveler changes after berth selection**: If passenger count changes, some berths may no longer be valid (e.g., lower berth only for 1 passenger). Recalculate availability and prompt user.

## Success Metrics

| Metric | Current State | Target | Measurement |
|--------|---------------|--------|------------|
| Berth selection loss during booking | 35% (resets) | <2% | Booking logs: selected_berth == final_berth |
| Mobile berth selection success rate | 60% | 95%+ | Mobile-only booking success rate |
| Berth unavailability discoveries post-booking | 8% | <0.5% | Support tickets: "Got different berth" |
| Time to complete berth selection step | 45 sec | 20 sec | UX testing: step entry → step exit |
| Elderly user (60+) berth booking success | 55% | 85%+ | User testing with 60+ age group |
| Satisfaction with berth selection clarity | 2.8/5 | 4.8/5 | Post-booking survey |

---

# Feature Spec 4: Persistent Session Management & Auto-Renewal

**Traceability:** [Part A — Problem 4: Repeated Login Requirement Causes Booking Delays](../part-a/PROBLEMS.md#problem-4-repeated-login-requirement-causes-booking-delays-self-discovered)

## Problem Statement

IRCTC repeatedly asks users to log in again even during active browsing sessions. Login popups interrupt the booking flow when users navigate between train search, reservation charts, and enquiry modules. This is especially problematic during Tatkal bookings when every second counts. The root cause is fragile session handling and lack of session token refresh logic. Users on unstable mobile networks experience even higher re-authentication rates.

## Proposed Solution

Implement robust session management with automatic token refresh, extended session lifetime, and silent re-authentication. Sessions remain valid for 4 hours of continuous activity (vs. current 30 minutes). Backend automatically refreshes the session token 15 minutes before expiry without requiring user action. Only if token refresh fails does the user see a login prompt—and even then, they can resume booking without losing progress.

## Technical Implementation Plan

### System Components Affected
1. Backend: JWT token management with refresh token rotation
2. Backend API: New `/auth/refresh` endpoint for silent token renewal
3. Frontend: Session interceptor middleware to auto-refresh tokens
4. Database: `session_tokens` table for token tracking and revocation
5. Auth service: Redis-based session store for fast lookups

### New Data Requirements

**`session_tokens` table:**
- `token_id` (UUID)
- `user_id` (FK)
- `access_token` (JWT, expires in 4 hours)
- `refresh_token` (long-lived JWT, expires in 30 days)
- `issued_at` (timestamp)
- `expires_at` (timestamp)
- `last_refreshed_at` (timestamp)
- `status` (active/revoked)
- `device_fingerprint` (to prevent token theft)

**`session_activity` (Redis, for fast read):**
- Key: `session:{user_id}:{device_id}`
- Value: `{ access_token, refresh_token, expires_at, activity_count }`
- TTL: 30 days (matches refresh token lifetime)

### API Changes

**1. POST `/auth/login` (modified)**
- Response now includes both `access_token` (4 hour lifetime) and `refresh_token` (30 day lifetime)
- Response:
  ```json
  {
    "access_token": "eyJhbGc...",
    "refresh_token": "eyJhbGc...",
    "expires_in": 14400,
    "refresh_expires_in": 2592000
  }
  ```

**2. POST `/auth/refresh` (new)**
- Request: `{ refresh_token }`
- Response: `{ access_token, expires_in }`
- Purpose: Silently refresh access token without user interaction
- Validation: Verify device fingerprint matches original login

**3. POST `/auth/logout` (modified)**
- Request: `{ refresh_token }`
- Purpose: Revoke both access and refresh tokens immediately
- Update `session_tokens.status = revoked`

**4. GET `/auth/validate-session` (new)**
- Response: `{ valid: true/false, remaining_time: seconds }`
- Purpose: Check if session is still valid (called on app foreground on mobile)

### Frontend State Changes

**Session Manager (Redux):**
```javascript
sessionState = {
  isAuthenticated: boolean,
  accessToken: string,
  refreshToken: string,
  expiresAt: timestamp,
  refreshExpiresAt: timestamp,
  user: { id, name, email, phone },
  
  // Session state
  lastActivityAt: timestamp,
  isRefreshing: boolean,
  refreshFailureCount: number
}
```

**Token Refresh Interceptor:**
- Before every API call, check: `now >= expiresAt - 15_minutes`
- If true, trigger `POST /auth/refresh` silently (no UI interruption)
- If refresh succeeds, update `accessToken` and `expiresAt`
- If refresh fails, either: (a) retry in 30 sec, or (b) show login prompt if retries exhausted
- If user navigates to login page anyway, show "Your session is still active" with "Continue browsing" button

**Mobile-specific enhancements:**
- On app foreground, call `GET /auth/validate-session` to verify session still valid
- On app background (5 min+), clear tokens from memory (keep only refresh token in secure storage)
- On app foreground again, automatically refresh using refresh token

### Third-party Services
- None (Redis is internal)

### Edge Cases and Constraints

1. **Refresh token expires while user is away**: Show "Your session expired. Please log in again" with minimal friction (pre-filled email).

2. **Network loss during token refresh**: Queue the refresh request. On reconnect, immediately attempt refresh before showing any UI.

3. **User logs in on multiple devices**: Each device gets its own refresh token. Sessions don't interfere. User can logout from one device independently.

4. **Attacker steals refresh token**: Include device fingerprint validation (User-Agent, IP, etc.). Token is rejected if device fingerprint changes. Alert user of potential breach.

5. **Session revocation (user reports device stolen)**: Revoke all tokens for user immediately via admin panel. User must login again on all devices.

6. **"Remember me for 30 days" feature (if implemented)**: Only extend refresh token to 30 days if user on secure device (HTTPS, trusted location). Revoke immediately if suspicious location detected.

7. **Booking in progress when session expires**: Before expiry, auto-refresh token. Booking continues seamlessly. If refresh fails mid-booking, bookmark their booking state and allow resume-after-login.

## Success Metrics

| Metric | Current State | Target | Measurement |
|--------|---------------|--------|------------|
| Unexpected login prompts during booking | 22% of sessions | <1% | Analytics: login_interrupt_events / total_bookings |
| Session timeout rate (premature) | 18% | <2% | Auth logs: premature_timeout_count |
| Token refresh success rate | N/A | 99%+ | `POST /auth/refresh` success_count / total_calls |
| Avg. time from login to confirmed booking | 240 sec | 180 sec | Analytics: confirm_booking_time - login_time |
| Session duration (continuous activity) | 30 min | 240 min | Session logs: session_end_time - session_start_time |
| Mobile app re-authentication rate | 35% | <5% | Mobile-only analytics |
| Booking abandonment due to login issues | 12% | <1% | Funnel analysis: session_expired / total_attempts |

---

# Feature Spec 5: PNR Enquiry Error Clarity & Smart Status Prediction

**Traceability:** [Part A — Problem 5: Correct PNR Sometimes Shows Wrong/Error Output](../part-a/PROBLEMS.md#problem-5-correct-pnr-sometimes-shows-wrongerror-output-self-discovered)

## Problem Statement

PNR enquiry system displays cryptic backend error messages like "FLUSHED PNR" or "PNR NOT YET GENERATED" without explanation. Users don't understand what these terms mean, assume their booking failed, and unnecessarily contact support or rebook tickets. The system exposes technical jargon instead of user-friendly status explanations. Waitlisted passengers are especially confused because they don't know if their PNR exists but hasn't updated yet, or if there's a real problem.

## Proposed Solution

Translate all backend PNR status messages into user-friendly explanations with actionable next steps. Show:
- Plain English status: "Your ticket is confirmed" vs "Your ticket is waitlisted"
- Why they're seeing an unclear message: "Your PNR was recently generated and railways are syncing data. Check again in 5 minutes."
- Prediction: For waitlisted tickets, show "Based on historical patterns, your WL 8 on this route usually confirms 2 days before departure"
- Next actions: "Share your PNR with family" or "Set a reminder to check on May 10"

## Technical Implementation Plan

### System Components Affected
1. Frontend: PNR status translator module (maps backend codes to user messages)
2. Backend API: Modified `/pnr/enquire` endpoint to include status translation
3. Database: `pnr_status_translations` table (mapping codes to messages)
4. ML/Analytics: WL confirmation predictor model (XGBoost)
5. Notification system: Reminder triggers for waitlisted bookings

### New Data Requirements

**`pnr_status_translations` table:**
- `status_code` (string: "FLUSHED_PNR", "NOT_YET_GENERATED", etc.)
- `user_facing_message` (string: Plain English explanation)
- `recommended_action` (string: What user should do)
- `retry_after_seconds` (integer: Suggest checking again in X seconds)
- `severity` (enum: "info" / "warning" / "error")

Example rows:
```
FLUSHED_PNR → "Your PNR has been processed but cleared from live database. Your ticket is confirmed on Indian Railways. → Download your ticket from email or call 139"

NOT_YET_GENERATED → "PNR created 10 minutes ago. Railways are generating your ticket in their system. → Check again in 15 minutes or search by booking reference instead"

WL_NOT_CONFIRMED → "Your ticket is on waiting list. Based on historical patterns for this route, your position usually confirms 2 days before. → Set a reminder for May 10"
```

**`waitlist_predictor_training_data` (training set for ML):**
- `route_id`, `date`, `class`, `wl_position_at_booking`, `wl_position_2days_before`, `final_status` (confirmed/not_confirmed)
- Historical data: 3+ years, 10M+ records

### API Changes

**1. GET `/pnr/enquire/{pnr_number}` (modified)**
- Response now includes:
  ```json
  {
    "pnr": "1234567890",
    "status": "WAITLISTED",
    "status_code": "WL_NOT_CONFIRMED",
    
    "user_friendly_message": "Your ticket is on waiting list #8. Based on historical patterns for this route, your position usually confirms 2 days before departure.",
    
    "prediction": {
      "type": "waitlist_confirmation",
      "probability": 0.78,
      "confidence": "high",
      "predicted_confirmation_date": "2026-05-13"
    },
    
    "recommended_actions": [
      "Set a reminder for May 10 to check confirmation status",
      "Share your PNR with family: 1234567890",
      "Download booking confirmation from email"
    ],
    
    "retry_suggestion": {
      "retry_after_seconds": 900,
      "reason": "Railways update PNR status every 15 minutes"
    },
    
    "passenger_details": { /* standard fields */ }
  }
  ```

**2. POST `/pnr/set-reminder` (new)**
- Request: `{ pnr_number, reminder_date }`
- Purpose: User can set reminder to check PNR status (triggers SMS/email/push)

**3. GET `/pnr/alternatives/{pnr_number}` (new)**
- Response: If PNR not found, suggest alternative trains on same route with available seats
- Purpose: Help user find backup option if PNR is genuinely lost

### Frontend State Changes

**PNR Status Translator Component:**
```javascript
// Maps backend codes to user-friendly interface
const statusTranslations = {
  "FLUSHED_PNR": {
    icon: "✅",
    color: "green",
    message: "Your ticket is confirmed on Indian Railways",
    actions: ["Download ticket", "View itinerary", "Contact support"]
  },
  "NOT_YET_GENERATED": {
    icon: "⏳",
    color: "blue",
    message: "PNR created 10 minutes ago. Railways are generating...",
    actions: ["Check again in 15 min", "Try searching by booking reference"]
  },
  "WL_NOT_CONFIRMED": {
    icon: "⏸",
    color: "orange",
    message: "Waitlist #8. Usually confirms 2 days before.",
    prediction: "78% chance of confirmation",
    actions: ["Set reminder", "Find backup train", "View alternatives"]
  }
}
```

**UI Enhancements:**
- Show status with color coding (green = confirmed, orange = waitlist, red = issue)
- Display prediction probability prominently for waitlisted bookings
- Add reminder button for key dates (2 days before departure)
- Show countdown: "Check again in 14:23"

### ML Model: Waitlist Confirmation Predictor

**Model Type:** XGBoost Classification (0 = Not Confirmed, 1 = Confirmed)

**Training Data:**
- Source: 3+ years historical IRCTC data (10M+ bookings)
- Features:
  - `route_id` (categorical)
  - `travel_date` (cyclical: day of week, week of year, holiday flag)
  - `class` (Sleeper, AC2, AC3, etc.)
  - `quota` (General, Tatkal, Senior Citizen)
  - `wl_position_at_booking` (numeric: 1–5000)
  - `days_until_departure` (numeric)
  - `season` (peak/off-peak)
  - `historical_confirmation_rate` (aggregate: this route usually has X% confirm rate)

**Model performance:**
- Accuracy: 82–87%
- Precision: 85%+ (avoid false positives: telling user they'll confirm when they won't)
- Fallback: If confidence <70%, don't show prediction—just show status

**Serving:**
- Wrap in Flask/FastAPI microservice
- Input: Query PNR details → Output: Confirmation probability
- Latency: <100ms per prediction

### Third-party Services
- None (all internal)

### Edge Cases and Constraints

1. **PNR number is invalid (user entered wrong digits)**: Show "PNR not found. Check spelling or try booking reference instead. Example: 1234567890"

2. **PNR exists but too old (3+ months)**: Show "This PNR is from January. To find your current booking, use recent PNR or search by date."

3. **User keeps checking same waitlisted PNR multiple times**: Show "We've set up automatic notifications. You don't need to check manually—we'll alert you if status changes."

4. **Prediction model is uncertain (confidence <60%)**: Show status without prediction: "We don't have enough historical data for this specific route. Check back 2 days before departure."

5. **Railways backend is down (no PNR data available)**: Show "Railways PNR system is temporarily unavailable. Your ticket is safe. Try again in 10 minutes."

6. **User queries PNR 1 hour after booking (still being processed)**: Proactively show "Your PNR was just created. Railways need 15–30 min to sync data. It'll show up automatically."

## Success Metrics

| Metric | Current State | Target | Measurement |
|--------|---------------|--------|------------|
| Support tickets for "unclear PNR status" | 15% of PNR enquiries | <2% | Support ticket categorization |
| PNR enquiry user satisfaction | 2.5/5 | 4.7/5 | Post-inquiry survey |
| Unnecessary rebooking due to unclear status | 18% | <2% | Booking analytics: rebookings within 1 hour |
| Waitlist prediction accuracy | N/A | 82%+ | ML model validation: predicted vs actual confirmation |
| Time spent on PNR enquiry page | 180 sec | 40 sec | Analytics: page session duration |
| Support contact avoidance (due to clarity) | N/A | 60% reduction in PNR-related calls | Support call logs |
| User confidence in PNR status | 3.1/5 | 4.8/5 | User testing survey |

---

# Feature Spec 6: Redesigned Reservation Chart with Information Hierarchy

**Traceability:** [Part A — Problem 6: Reservation Charts Are Cluttered and Difficult to Understand](../part-a/PROBLEMS.md#problem-6-reservation-charts-are-cluttered-and-difficult-to-understand-self-discovered)

## Problem Statement

The current reservation chart displays dense tables with weak information hierarchy, poor visual grouping, and minimal color differentiation. Users must manually inspect rows to find available berths, which is exhausting on mobile devices and for elderly users. The interface doesn't differentiate between berth availability, blocked berths, and already-booked seats. Critical information (e.g., "Lower berth, Coach A, Window side") is buried in small text alongside irrelevant metadata.

## Proposed Solution

Redesign reservation chart as a visual berth grid (not a table) with:
1. **Color-coded berth status** (available = green, booked = red, blocked = grey)
2. **Visual berth layout** showing actual train coach structure (8 berths per side in Sleeper)
3. **Smart filtering** (show only available / show all / filter by berth type)
4. **Coach carousel** to navigate through coaches without scrolling
5. **Touch-friendly berth selection** (large tap targets on mobile)
6. **Info panel** showing selected berth details (price, preferences, restrictions)

## Technical Implementation Plan

### System Components Affected
1. Frontend: New React component `ReservationChart` with visual grid layout
2. Frontend: Canvas or SVG rendering for berth grid visualization
3. Backend API: New `/reservation-chart/visual` endpoint returning berth data in visual format
4. Database: Cached berth availability data (updated real-time)
5. Real-time service: WebSocket updates for live berth availability changes

### New Data Requirements

**`berth_visual_layout` table (configuration):**
- `train_id` (FK)
- `class_type` (Sleeper, AC2, etc.)
- `coach_layout` (JSON: describing berth positions, preferences, restrictions)
- Example:
  ```json
  {
    "berths_per_side": 8,
    "coach_structure": [
      {
        "position": 1,
        "type": "Lower",
        "side": "left",
        "accessibility": ["window", "aisle_adjacent"]
      },
      {
        "position": 2,
        "type": "Middle",
        "side": "left",
        "accessibility": []
      }
    ]
  }
  ```

**`berth_availability_cache` (Redis, updated real-time):**
- Key: `chart:{train_id}:{class}:{date}`
- Value: `{ berths: [{position, status, price, blocked_reason}], last_updated }`
- TTL: 2 minutes (expires if not refreshed)

### API Changes

**1. GET `/reservation-chart/visual/{train_id}/{class}` (new)**
- Query params: `date`, `quota` (General/Tatkal), `coach_filter` (optional)
- Response:
  ```json
  {
    "train_id": "12622",
    "class": "Sleeper",
    "date": "2026-05-15",
    "total_berths": 72,
    "available_berths": 12,
    "coaches": [
      {
        "coach_number": "A1",
        "position": 1,
        "berths": [
          {
            "berth_id": 1,
            "position": 1,
            "type": "Lower",
            "side": "left",
            "status": "available",
            "price": 500,
            "restrictions": []
          },
          {
            "berth_id": 2,
            "position": 2,
            "type": "Middle",
            "side": "left",
            "status": "booked",
            "booked_by": "PNR1234"
          }
        ],
        "coach_availability": "5/8"
      }
    ],
    "last_updated": "2026-05-08T14:32:10Z",
    "next_refresh_at": "2026-05-08T14:34:10Z"
  }
  ```

**2. GET `/reservation-chart/coaches` (new, for carousel)**
- Response: List of all coaches with availability summary
- Purpose: Coach carousel navigation (don't load all coaches, paginate)

**3. POST `/reservation-chart/subscribe` (new, WebSocket)**
- Purpose: Real-time updates when berth status changes
- Message: `{ berth_id, new_status, timestamp }`

### Frontend State Changes

**Reservation Chart Manager:**
```javascript
chartState = {
  trainId: "12622",
  classType: "Sleeper",
  date: "2026-05-15",
  
  // Visual data
  coaches: [
    {
      coachNumber: "A1",
      berths: [
        { berthId: 1, type: "Lower", status: "available", price: 500 },
        { berthId: 2, type: "Middle", status: "booked" },
        ...
      ]
    }
  ],
  
  // Filter & interaction
  activeCoach: "A1",
  filterBy: "all", // all / available_only / berth_type (Lower/Middle/Upper)
  selectedBerth: null,
  lastUpdated: timestamp
}
```

**UI Components:**
- `CoachCarousel`: Horizontal scrollable coach selector (mobile-friendly)
- `BerthGrid`: Visual 8x2 grid showing all berths in a coach
- `BerthCard`: Individual berth tile (color-coded status + price)
- `BerthDetail`: Panel showing selected berth info (preferences, price, booking flow)
- `AvailabilityFilter`: Dropdown to filter berths

**Mobile-specific UI:**
- Berth tiles are 50x50px (large tap targets)
- Coach carousel uses horizontal swipe (not buttons)
- Zoom capability: pinch-to-zoom on berth grid
- Show only one coach at a time (carousel)

**Desktop UI:**
- Show 2–4 coaches side-by-side
- Berth tiles 40x40px
- Traditional vertical scrolling

### Real-time Updates

**WebSocket subscription on page load:**
```javascript
socket.on(`chart:${trainId}:${date}`, (message) => {
  // Update single berth in UI
  updateBerthStatus(message.berth_id, message.new_status);
  // Smooth animation (fade berth color change)
});
```

### Third-party Services
- None (WebSocket is internal)

### Edge Cases and Constraints

1. **Mobile network is very slow (2G)**: Load only 1 coach at a time. Pre-cache next/prev coaches in background.

2. **Berth becomes unavailable while user is viewing chart**: Highlight the berth in red + show notification: "This berth was just booked. Choose another?"

3. **Real-time updates flood in (100+ berth changes/second during peak)**: Batch updates into 1-second chunks to avoid UI thrashing.

4. **User has accessibility needs (color-blind, screen reader)**: Use patterns (stripes, dots) in addition to colors. Add ARIA labels for each berth.

5. **Legacy browsers don't support WebSocket**: Fallback to polling (fetch chart every 3 seconds).

6. **User selects a berth then page reloads**: Store selected berth in localStorage. On reload, pre-select it (if still available).

7. **Train has unusual berth layout (e.g., some coaches are all-women or RAC-only)**: Show coach metadata badges ("All-Women Coach", "RAC only", "AC with side lower").

## Success Metrics

| Metric | Current State | Target | Measurement |
|--------|---------------|--------|------------|
| Time to find available berth | 120 sec | 20 sec | User testing: chart load → berth selection |
| Elderly (60+) user chart comprehension | 35% | 90%+ | User testing with 60+ group |
| Mobile user satisfaction with chart | 2.1/5 | 4.6/5 | Post-booking survey (mobile only) |
| Berth selection errors (wrong berth chosen) | 14% | <1% | Booking logs: selected != final berth |
| Chart page load time | 3.2 sec | <1.2 sec | WebVitals: LCP metric |
| Accessibility compliance score | 45% (WCAG) | 95%+ (WCAG AA) | Automated accessibility audit |
| Support tickets about reservation charts | 8% of traffic | <0.5% | Support categorization |
| User confidence in berth availability info | 3.0/5 | 4.9/5 | User testing survey |

---

## Traceability Matrix

| Feature Spec | Part A Problem | Problem Statement | Affected Users | Impact (Peak) |
|--|--|--|--|--|
| 1. Tatkal Queue | Problem 1 | Platform crashes during Tatkal opening | 20–40 lakh daily | Critical (affects primary use case) |
| 2. Search Filters | Problem 2 | Filters reset, stale data shown | All search users | High (affects all bookings) |
| 3. Berth State | Problem 3 | Selected berth resets mid-flow | 30–40% of users | High (completion blocker) |
| 4. Session Management | Problem 4 | Repeated login interrupts flow | Millions daily | High (UX friction) |
| 5. PNR Clarity | Problem 5 | Unclear error messages confuse users | Millions of enquiries | Medium (causes support load) |
| 6. Chart Redesign | Problem 6 | Dense UI difficult to scan | Thousands per release | Medium (affects discovery speed) |

---

## Next Steps

1. **Wireframes** (documented in WIREFRAMES.md) show UI mockups for specs 1–6
2. **AI Feature** (documented in AI-FEATURE.md) proposes waitlist confirmation predictor
3. **2×2 Matrix** (documented in MATRIX.md) prioritizes these 6 specs by impact/effort
4. **Peer Review** section collected feedback and updated specs accordingly

