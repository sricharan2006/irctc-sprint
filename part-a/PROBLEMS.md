# IRCTC Problem Discovery — Part A

## Summary

- Total problems documented: 6
- Given problems: 3
- Self-discovered problems: 3
- Platform explored: irctc.co.in
- Exploration type: Live testing
- Devices used: Windows Laptop + Chrome Browser
- Exploration date: 07 May 2026

---

# Problem 1: Tatkal Booking Crashes at 10:00 AM [Given]

## What is broken

The IRCTC platform becomes extremely slow or completely unresponsive during Tatkal booking opening time at 10:00 AM. Users experience loading freezes, HTTP 502 errors, CAPTCHA resets, session timeouts, and payment uncertainty during the most important booking window.

The platform fails to provide queue visibility, progress tracking, or proper feedback while the request is being processed.

---

## Affected users

- Tatkal passengers
- Daily commuters
- Emergency travelers
- Tier 2 and Tier 3 city users
- Users with slower internet connections

Estimated impact:
20–40 lakh users actively attempting Tatkal bookings between 9:58 AM and 10:05 AM daily.

---

## Frequency

Occurs daily during Tatkal booking opening hours.

This is a long-standing and publicly known issue.

---

## Current flow — step by step

1. User opens IRCTC website around 9:50 AM
2. User logs into account
3. User searches for train and selects Tatkal quota
4. User fills passenger details before 10:00 AM
5. User clicks “Book Now” exactly near 10:00 AM
6. Platform freezes or shows continuous loading spinner
7. User waits 15–45 seconds without feedback
8. Website throws HTTP 502 error or session timeout
9. CAPTCHA resets or user gets logged out
10. User logs in again and sees Tatkal quota already filled

---

## Where exactly it breaks

Steps 6–9:
The platform cannot handle massive concurrent requests during Tatkal opening. Lack of queue management and feedback systems increases panic refreshes, which further overloads servers.

---

# Problem 2: Search Filters Do Not Work Reliably [Given]

## What is broken

Train search filters for class type, quota, availability, and departure timing often fail to work correctly. Filters sometimes reset automatically after refreshes or display trains that do not match the selected conditions.

This forces users to manually search through large train lists repeatedly.

---

## Affected users

- Daily train passengers
- Senior citizens
- First-time IRCTC users
- Users searching during high traffic hours

Estimated impact:
Potentially affects all train search users across the platform.

---

## Frequency

Occurs intermittently but frequently during train searches.

More noticeable during high traffic periods.

---

## Current flow — step by step

1. User searches trains between two stations
2. Platform displays multiple train results
3. User applies filters such as:
   - Sleeper Class
   - Available seats only
   - Morning departure
4. Page refreshes
5. Waitlisted trains still appear
6. User opens a train expecting availability
7. Platform shows waitlist instead
8. User clicks back
9. Previously selected filters reset automatically
10. User manually searches again

---

## Where exactly it breaks

Steps 4–9:
The filter state is not preserved properly and stale availability data appears after refreshes.

---

# Problem 3: Seat Selection Resets Randomly [Given]

## What is broken

Selected berth preferences disappear during the booking process. Users who select specific berths such as lower berths often find that the system changes the preference to “Auto” or assigns another berth.

This issue is more common on mobile devices.

---

## Affected users

- Elderly passengers
- Families traveling together
- Users requiring lower berths
- Mobile users
- Passengers with physical difficulties

Estimated impact:
30–40% of users attempting berth selection.

---

## Frequency

Occurs frequently during booking flows involving seat selection.

Higher occurrence rate on mobile devices.

---

## Current flow — step by step

1. User selects train and class
2. User opens berth selection interface
3. User selects preferred berth
4. Selected berth gets highlighted
5. User clicks Proceed
6. Passenger details page opens
7. Preferred berth changes to Auto or another berth
8. User goes back to check selection
9. Original berth becomes unavailable
10. User proceeds with incorrect berth assignment

---

## Where exactly it breaks

Steps 5–7:
Seat selection state is not preserved correctly between booking flow components.

---

# Problem 4: Repeated Login Requirement Causes Booking Delays [Self-Discovered]

## What is broken

IRCTC repeatedly asks users to log in again even during active browsing sessions. Login popups appear while navigating between train search, reservation charts, and enquiry modules.

This creates unnecessary delays and interrupts user flow during important booking activities.

---

## Affected users

- Tatkal booking users
- Frequent IRCTC users
- Users switching between tabs/modules
- Mobile users with unstable internet
- Senior citizens unfamiliar with repeated authentication

Estimated impact:
Lakhs of users daily during booking and enquiry sessions.

---

## Frequency

Observed multiple times during live testing.

Occurs frequently during:
- Session refreshes
- Navigation between modules
- Extended browsing sessions

---

## How I found it

While navigating between reservation charts and enquiry modules, the platform repeatedly triggered login popups despite active usage.

Screenshot:
`assets/screenshots/login-delay.png`

---

## Current flow — step by step

1. User opens IRCTC website
2. User logs into account
3. User searches for trains
4. User opens another service page such as reservation charts
5. Platform unexpectedly asks user to log in again
6. User re-enters username, password, and CAPTCHA
7. Website reloads slowly
8. Previous browsing flow gets interrupted
9. User loses time during booking process
10. User repeats navigation again

---

## Where exactly it breaks

Step 5:
The platform fails to maintain stable authentication sessions across multiple IRCTC modules.

---

# Problem 5: Correct PNR Sometimes Shows Wrong/Error Output [Self-Discovered]

## What is broken

The PNR enquiry system sometimes displays unclear backend-related messages such as:

- “FLUSHED PNR”
- “PNR NOT YET GENERATED”

even when users enter seemingly correct PNR numbers.

The platform does not explain what these messages mean or what users should do next.

---

## Affected users

- Waitlisted passengers
- Users checking ticket confirmation
- Elderly passengers
- First-time railway travelers
- Users unfamiliar with railway terminology

Estimated impact:
Lakhs of daily PNR enquiry users.

---

## Frequency

Occurs intermittently depending on:
- Server synchronization delays
- Newly generated PNRs
- High traffic conditions

Observed during live testing.

---

## How I found it

I entered a valid-looking PNR in the enquiry system and received an unclear technical error message instead of proper ticket information.

Screenshot:
`assets/screenshots/pnr-error.png`

---

## Current flow — step by step

1. User opens Indian Railways PNR enquiry page
2. User enters PNR number
3. User clicks Submit
4. System processes request
5. Error message appears:
   “FLUSHED PNR / PNR NOT YET GENERATED”
6. No explanation or recovery guidance appears
7. User retries multiple times
8. User becomes unsure whether ticket exists or failed
9. User may contact support unnecessarily
10. User loses trust in the enquiry system

---

## Where exactly it breaks

Step 5:
The system exposes technical backend messages instead of user-friendly status explanations.

---

# Problem 6: Reservation Charts Are Cluttered and Difficult to Understand [Self-Discovered]

## What is broken

The reservation chart interface is visually cluttered and difficult to scan quickly. Important berth details, coach information, and station mapping are densely packed into large tables with weak readability.

The platform lacks:
- Proper information hierarchy
- Visual grouping
- Clear berth indicators
- Color differentiation
- Mobile-friendly readability

Users must manually inspect rows repeatedly to understand berth availability.

---

## Affected users

- Elderly passengers
- First-time IRCTC users
- Mobile users
- Last-minute travelers
- Users checking berth availability quickly

Estimated impact:
Thousands of users during every reservation chart release cycle.

---

## Frequency

Always reproducible.

Observed consistently during live testing.

---

## How I found it

I opened the reservation chart page for Train No. 12797 and observed that berth details were displayed in a dense table layout with poor readability and difficult navigation.

Screenshot:
`assets/screenshots/reservation-chart-confusing.png`

---

## Current flow — step by step

1. User opens reservation chart page
2. User searches for train chart
3. Reservation chart table loads
4. User tries to identify berth availability
5. Multiple dense columns appear together
6. Important information becomes difficult to scan
7. User repeatedly checks rows manually
8. Mobile readability becomes worse
9. User struggles to compare berth details quickly
10. User spends extra time understanding basic information

---

## Where exactly it breaks

Steps 4–7:
The interface lacks proper information hierarchy and visual grouping, causing cognitive overload during berth analysis.

---

# Overall Key Observations

The IRCTC platform suffers from multiple large-scale UX and system-level issues including:

- Traffic scalability problems
- Poor feedback systems
- Weak session handling
- Inconsistent state management
- Information overload
- Accessibility limitations
- Poor mobile optimization

These issues directly affect millions of passengers daily and increase:
- Booking anxiety
- Failed ticket attempts
- Passenger confusion
- Extra booking time
- User frustration

---

# Future Scope — Part B

The findings from Part A can be used in Part B for:

- UX redesign proposals
- Queue management systems
- AI-assisted passenger support
- Smart availability systems
- Better session handling
- Accessibility-focused redesigns
- Improved reservation chart visualization
- Performance optimization architecture

---

# Conclusion

This document presents six real IRCTC passenger pain points identified through live platform exploration and structured UX analysis.

The documented problems provide a strong foundation for proposing scalable, user-centered, and technically feasible improvements in Part B of the IRCTC Design Engineering Sprint.