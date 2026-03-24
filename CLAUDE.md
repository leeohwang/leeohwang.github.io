# CLAUDE.md — Tally iOS App

## Project Overview

Tally is an iOS app that answers one question daily: "Am I making progress, or am I standing still?" Users define 3-5 personal "Life Tracks" (things that make a good week), then do a 20-second nightly check-in rating each track as Moved (green), Held (neutral), or Drifted (amber). Weekly reports reveal patterns, drift alerts, and momentum streaks. Freemium model: free daily check-ins, paid weekly/monthly insights ($5.99/mo).

## Tech Stack

- **Language:** Swift
- **UI Framework:** SwiftUI
- **Persistence:** SwiftData (preferred) or Core Data
- **Minimum Target:** iOS 17+
- **Architecture:** MVVM
- **No external dependencies** unless absolutely necessary

## Project Structure (Target)

```
Tally/
├── TallyApp.swift                  # App entry point
├── Models/                         # SwiftData models
│   ├── LifeTrack.swift
│   ├── DailyCheckIn.swift
│   └── WeeklyReport.swift
├── ViewModels/                     # Business logic
│   ├── SetupViewModel.swift
│   ├── CheckInViewModel.swift
│   ├── DashboardViewModel.swift
│   └── WeeklyReportViewModel.swift
├── Views/
│   ├── Setup/                      # Onboarding flow
│   │   ├── OpeningView.swift
│   │   ├── TrackInputView.swift
│   │   └── TrackRevealView.swift
│   ├── CheckIn/                    # Daily check-in
│   │   ├── CheckInView.swift
│   │   ├── TrackRowView.swift
│   │   └── CompletionView.swift
│   ├── Dashboard/                  # Home screen
│   │   ├── DashboardView.swift
│   │   ├── CalendarView.swift
│   │   ├── DriftAlertCard.swift
│   │   └── StreakDisplay.swift
│   ├── WeeklyReport/               # Week in Motion (paid)
│   │   ├── WeeklyReportView.swift
│   │   ├── MovementScoreView.swift
│   │   └── MotionMapView.swift
│   └── Common/                     # Reusable components
│       ├── SlotMachineTextField.swift
│       └── Theme.swift
├── Services/
│   ├── StreakService.swift
│   ├── DriftDetectionService.swift
│   ├── MovementScoreService.swift
│   └── NotificationService.swift
├── Utilities/
│   └── DateHelpers.swift
└── Assets.xcassets/
```

## Design Language

- **Color palette:** Near-black backgrounds, warm whites, muted greens/ambers/grays
- **Moved:** Muted organic green (not neon)
- **Held:** Warm gray/sand
- **Drifted:** Muted amber/ochre
- **No red anywhere** — gray for lowest scores, never punishing
- **Typography:** Clean humanist sans-serif, generous spacing
- **Animations:** All under 400ms, ease-out or spring physics, never linear
- **Tone:** Dignified, honest, no gamification, no toxic positivity
- **Haptics:** Single firm tap for Moved, none for Held, two soft taps for Drifted

## Key Design Rules

1. The app is a mirror, not a coach — it observes, never instructs
2. No streak guilt, no "you failed" messaging, no confetti
3. No "Submit" button on check-in — completion is automatic when all rows resolved
4. Completion messages are contextual and honest (e.g., "You noticed. That's the whole point.")
5. The daily check-in must FEEL like 20 seconds. If it feels longer, the design has failed
6. Free tier lets users build the habit; paid tier reveals the patterns

---

## Task Breakdown

### Phase 1: Foundation & Data Layer
> Get the skeleton right before any UI polish.

**Task 1.1 — Data Models**
- Create `LifeTrack` model (id, name, displayOrder, createdAt, isPaused)
- Create `DailyCheckIn` model (id, date, trackId, status enum [moved/held/drifted], note?)
- Create `WeeklyReport` model (id, weekStartDate, movementScore, isSaved)
- Set up SwiftData container and model configuration in TallyApp.swift

**Task 1.2 — App Navigation Shell**
- Implement root navigation: check if setup is complete → show Setup or Dashboard
- Use NavigationStack for setup flow, TabView or single-screen for main app
- Store `hasCompletedSetup` in UserDefaults or AppStorage

**Task 1.3 — Theme & Design System**
- Create `Theme.swift` with all colors, fonts, spacing constants
- Define the three status colors (moved green, held gray, drifted amber)
- Create reusable color/font modifiers

---

### Phase 2: Setup / Onboarding Flow
> The first 120 seconds of the app. No account creation, no email — just the question.

**Task 2.1 — Opening Screen**
- Near-black background with slow radial pulse animation (heartbeat)
- Text appears word-by-word with timed pauses
- "Most apps ask what you want to do." → pause → "This one asks what makes your life feel like it's actually moving."
- Subtle downward indicator to proceed

**Task 2.2 — Slot Machine Text Field**
- Build `SlotMachineTextField` component: vertical cycling placeholder text
- Smooth translateY animation, 1800ms hold + 600ms transition, 0.3 alpha warm gray
- Stops on tap (300ms exit), resumes on blur after 1.5s delay
- Pool of ~24 curated cycling strings, Fisher-Yates shuffle per session
- Each track input gets a different randomized subset

**Task 2.3 — Track Input Flow**
- "Track 1 of 5" label, full-width input, warm white text ~20px
- On submit: text lifts and locks above input with checkmark, haptic pulse
- After 3 tracks: show "That's enough" text link
- After 5 tracks: auto-close input
- 80-char invisible limit (font compresses, never truncates)

**Task 2.4 — Track Reveal Screen**
- Staggered slide-in animation for each track (400ms each, from right, scale 0.95→1.0)
- Each track as horizontal card with name + color dot + empty progress line
- Contextual text: "These are your Life Tracks." → "Not goals. Not habits..." → "You can change them anytime."
- "Begin" button with slow border-glow animation
- Persist tracks to SwiftData, mark setup as complete

---

### Phase 3: Daily Check-In
> The core 20-second ritual. This is the product.

**Task 3.1 — Check-In Screen Layout**
- Dark, warm background (nighttime reflection feel)
- Each Life Track as full-width row: name left, three tappable states right
- States: Moved (green chevron up), Held (gray dash), Drifted (amber chevron down)
- All states visible but dimmed by default, no pre-selection

**Task 3.2 — Row Interaction & Animation**
- On tap: selected state scales 1.0→1.05 (150ms), color saturates, others fade to 0.15 alpha
- Underline sweep left-to-right (400ms) on resolution
- Left-border glow extinguishes on resolution
- Re-tap to change: spring animation re-expands all three options
- No enforced order, but visual weight shifts downward

**Task 3.3 — Optional +1 Sentence Field**
- Small pencil icon at 0.25 alpha appears after tapping a state
- Tapping opens single-line input below the row
- Cycling placeholder: "One sentence, if you want..." / "What happened?" / "The short version..."
- On submit: collapses, tiny quotation mark icon remains
- Save note to DailyCheckIn model

**Task 3.4 — Completion Sequence**
- Auto-detect when all rows are resolved (no Submit button)
- 500ms pause → rows compress upward → contextual message fades in:
  - Mostly Moved: "Good day. They add up."
  - Mostly Held: "Holding is fine. Not every day is a push day."
  - Mostly Drifted: "You noticed. That's the whole point."
  - Mixed: "Real days look like this."
- Date stamp at 0.4 opacity below message
- Hold 2s, then swipe-down dismissal or auto-fade
- Persist all check-in data

**Task 3.5 — Haptic Feedback**
- Moved: single firm haptic (UIImpactFeedbackGenerator .medium)
- Held: no haptic
- Drifted: two soft taps (UIImpactFeedbackGenerator .light × 2)

---

### Phase 4: Dashboard / Home Screen
> What the user sees every time they open the app (after setup).

**Task 4.1 — Dashboard Layout**
- Show today's check-in status (or prompt to check in)
- Display Life Tracks with today's status as a simple daily card
- Navigation to check-in screen

**Task 4.2 — Basic Calendar View (Free Tier)**
- Month grid showing green/neutral/amber dots per day
- Each dot represents the dominant status of that day's check-in
- Tappable days to see detail
- Scrollable month-to-month

**Task 4.3 — Streak Display**
- Calculate momentum streak: consecutive weeks where every track has >=1 moved day
- Dot chain visualization (horizontal dots, one per week)
- Color progression: neutral (weeks 1-4) → richer (5-12) → golden (13+)
- Current week heartbeat: show which tracks have been moved this week
- "N of M tracks moved — X to go this week"

**Task 4.4 — Drift Alert Card**
- Detect tracks with 14+ consecutive amber days
- Display drift alert card with factual message (never guilt)
- Tier 1: simple "drifting for N days"
- Tier 2: "longest drift since you started"
- Tier 3: pattern detection ("tends to happen in [month]")
- Snooze (7 days) and View History actions
- Max 1 drift alert per day, no alerts during check-in

---

### Phase 5: Weekly Report (Paid Feature)
> The core paid value — "Week in Motion" report.

**Task 5.1 — Movement Score**
- Calculate: % of track-days rated as "moved" across the week
- Large centered percentage with count-up animation (0% → final)
- Color gradient: 75-100% green, 50-74% teal, 25-49% amber, 0-24% gray
- Contextual line: "You moved forward on X of Y tracks this week"
- Week-over-week trend arrow (up/down/same)

**Task 5.2 — Motion Map**
- Horizontal bar per Life Track, 7 segments (Mon-Sun)
- Each segment colored by that day's tap (green/neutral/amber/empty)
- Tracks ordered by most motion → least motion (top → bottom)
- Tap-to-expand: shows per-day details and streak summary
- Must fit on one screen — no scrolling

**Task 5.3 — Weekly Insight**
- Algorithmically select ONE most emotionally relevant insight
- Examples: "You haven't missed Health in 4 weeks" / "Creative Work moved for the first time in 3 weeks"
- Only one insight per report — restraint is the design

**Task 5.4 — Sunday Notification & Reveal**
- Schedule local notification Sunday evening
- Rotating warm copy: "Your week just told its story." / "See how far you moved."
- Report unveils with brief animation (motion blur → clarity)
- Darker, more "cinematic" background for reflection mode

---

### Phase 6: Paywall & Monetization

**Task 6.1 — Paywall Screen**
- Trigger after 7 days of free check-ins (when first weekly report is ready)
- Show blurred/teaser version of the weekly report behind paywall
- Clear value prop: "See whether your week actually moved"
- Two tiers: $5.99/month or $34.99/year

**Task 6.2 — StoreKit Integration**
- Implement StoreKit 2 for in-app subscriptions
- Handle purchase, restore, and subscription status
- Gate premium features: weekly report, trends, drift timeline, streak history, milestones

---

### Phase 7: Notifications & Polish

**Task 7.1 — Daily Check-In Reminder**
- Default 8:45 PM notification
- Rotating copy from curated set (5-6 options)
- User-configurable time in settings
- Understated tone — ritual, not guilt

**Task 7.2 — Settings Screen**
- Edit/reorder/pause Life Tracks
- Change notification time
- Manage subscription
- Disable drift alerts per track
- Export data

**Task 7.3 — Missed Day Handling**
- No guilt on missed days — the app says nothing
- Empty days show as faint outline in calendar
- Streaks handle missed days via weekly (not daily) calculation

**Task 7.4 — Animation Polish Pass**
- Verify all animations match spec (slot machine, check-in resolution, completion, reveal)
- Ensure haptics are correct
- Test all timing and easing curves
- Dark mode color adjustments (luminous greens, warmer ambers)

---

### Phase 8: Future Features (Post-Launch)

- Monthly and quarterly trend views
- "This Time Last Month" comparisons
- Year in Review
- Drift Timeline ("River" visualization)
- Milestone celebrations with identity-affirming copy
- Grace mechanic (1-track save per streak)
- Home screen widget with ambient glow
- Share mechanic (blurred Motion Map image)

---

## Build & Run

```bash
# Open in Xcode
open Tally.xcodeproj

# Build via command line
xcodebuild -project Tally.xcodeproj -scheme Tally -destination 'platform=iOS Simulator,name=iPhone 16' build
```

## Important Reminders

- Always test that check-in feels like ~20 seconds end-to-end
- Never add gamification (badges, points, leaderboards)
- Keep completion messages honest and dignified — no "Great job!" or confetti
- The word "Drifted" is not failure — it's awareness
- Slot machine strings include raw/honest entries ("Beating lust", "Staying sober") — this is intentional, do not sanitize
- No social features, no AI suggestions, no integrations — the app is a mirror, not a coach
