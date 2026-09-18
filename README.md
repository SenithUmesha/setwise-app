<p align="center">
  <img src="https://setwise.senithumesha.com/images/setwise/app-icon.png" width="112" alt="SetWise app icon" />
</p>

<h1 align="center">SetWise</h1>

<p align="center">
  <strong>Train with a plan. Learn from every session.</strong>
</p>

<p align="center">
  A native iPhone & iPad workout tracker that connects the work you log with recovery, progress, and the next session.
</p>

<p align="center">
  <a href="https://apps.apple.com/us/app/setwise-workout-recovery/id6796000747">App Store</a>
  ·
  <a href="https://setwise.senithumesha.com/">Website</a>
  ·
  <a href="https://setwise.senithumesha.com/privacy">Privacy</a>
  ·
  <a href="https://setwise.senithumesha.com/support">Support</a>
  ·
  <a href="docs/engineering.md">Engineering notes</a>
</p>

<p align="center">
  <code>SwiftUI</code> · <code>SwiftData</code> · <code>WidgetKit</code> · <code>App Intents</code> · <code>StoreKit</code> · <code>offline-first</code>
</p>

<p align="center">
  <a href="https://github.com/SenithUmesha/setwise-app/actions/workflows/docs-check.yml"><img src="https://github.com/SenithUmesha/setwise-app/actions/workflows/docs-check.yml/badge.svg" alt="Docs integrity" /></a>
</p>

---

## why i built it

Most workout apps are good at recording sets. What I wanted was the part after that too: _what did this session actually change, what has been recovering, and what should I know before I train again?_

So SetWise became a pretty simple loop:

```text
plan a workout
      ↓
log the sets you actually complete
      ↓
update recovery + training history
      ↓
review progress
      ↓
come back with context for the next session
```

No social feed. No AI coach pretending to know your body. No account required just to log a workout.

It is intentionally a **private, local-first iOS app** built around the gym session itself.

---

## the app

<table>
  <tr>
    <td align="center"><img src="https://setwise.senithumesha.com/images/setwise/release-2/workout-home.png" width="210" alt="SetWise Today screen" /></td>
    <td align="center"><img src="https://setwise.senithumesha.com/images/setwise/release-2/saved-routine.png" width="210" alt="SetWise saved workout routine" /></td>
    <td align="center"><img src="https://setwise.senithumesha.com/images/setwise/release-2/active-workout.png" width="210" alt="SetWise active workout" /></td>
    <td align="center"><img src="https://setwise.senithumesha.com/images/setwise/release-2/calendar.png" width="210" alt="SetWise workout calendar" /></td>
  </tr>
  <tr>
    <td align="center"><sub>today</sub></td>
    <td align="center"><sub>plan</sub></td>
    <td align="center"><sub>track</sub></td>
    <td align="center"><sub>history</sub></td>
  </tr>
</table>

### plan

Build reusable workout programs around the way you actually train.

- fixed-weekday and rotating plans
- exercise prescriptions with target sets and rep ranges
- one-tap access to the next scheduled session
- saved routines you can keep tweaking instead of rebuilding every week

### track

The active workout screen is designed to stay useful between sets, not become another thing to fight with in the gym.

- previous weight + reps kept close to the current set
- warm-up and working sets in the same flow
- optional RIR tracking
- automatic rest timer with quick adjustments
- plate calculator without leaving the workout
- exercise guides with reviewed demo media when available
- active-session persistence so leaving the app does not throw away the workout

### review

Finishing a workout updates the context around your training automatically.

- completed-session history
- gym-day calendar
- exercise progress charts
- muscle-level recovery estimates
- weighted muscle distribution
- monthly reports
- yearly training recaps

Recovery is a **training signal**, not a medical claim. It is derived from the work you log and is there to give the next session some context, not tell you whether you are injured or guarantee readiness.

---

## useful without turning into a subscription maze

The core workout experience stays available for free: planning, workout tracking, set logging, exercise guides, recent history, the current calendar, rest timer, plate calculator, recovery context, and local data export.

**SetWise Pro** is a single lifetime in-app purchase that unlocks the deeper review layer: unlimited plans, the full training archive, longer progress ranges, muscle distribution, monthly reports, yearly recaps, recap sharing, and Home Screen widgets.

No recurring subscription.

---

## the fun iOS stuff

SetWise was also my excuse to build a product properly around Apple platform features instead of treating iOS as just another deployment target.

```text
SwiftUI               screens + interaction
SwiftData              local workout/program/history store
WidgetKit              recovery, calendar, streak + next-workout widgets
App Intents            Shortcuts / system actions
App Groups             lightweight widget snapshot sharing
UserNotifications      local reminders + recovery-ready notifications
BGTaskScheduler        opportunistic background refresh
StoreKit               lifetime Pro purchase + restore flow
XCTest / UI tests      domain + workout-flow verification
Privacy manifest       explicit platform privacy configuration
```

The production app has **no user account, no workout-data backend, no ads, and no in-app analytics SDK**. Training data stays on-device. Exercise demo media is the main online content path and is fetched only when needed.

If you're interested in the implementation choices rather than the product tour, I wrote them up in **[docs/engineering.md](docs/engineering.md)**.

---

## some of the deeper screens

<table>
  <tr>
    <td align="center"><img src="https://setwise.senithumesha.com/images/setwise/release-2/progress-analytics.png" width="230" alt="SetWise progress analytics" /></td>
    <td align="center"><img src="https://setwise.senithumesha.com/images/setwise/release-2/muscle-insights.png" width="230" alt="SetWise muscle insights" /></td>
    <td align="center"><img src="https://setwise.senithumesha.com/images/setwise/release-2/monthly-report.png" width="230" alt="SetWise monthly report" /></td>
    <td align="center"><img src="https://setwise.senithumesha.com/images/setwise/release-2/training-recap.png" width="230" alt="SetWise yearly training recap" /></td>
  </tr>
  <tr>
    <td align="center"><sub>progress</sub></td>
    <td align="center"><sub>muscle insights</sub></td>
    <td align="center"><sub>monthly report</sub></td>
    <td align="center"><sub>training recap</sub></td>
  </tr>
</table>

---

## a few decisions i care about

**local data is the source of truth**  
Workouts, programs, set history, settings, and recovery inputs live on the device. The app does not need a server round-trip to tell you what you did five minutes ago.

**recovery is calculated, not stored as fake truth**  
The app derives recovery from source data and current time. A percentage can change as time passes; it should not become a permanently stored fact just because it was once calculated at 62%.

**widgets get a snapshot, not the whole app database**  
The main app writes a small App Group snapshot for the widget extension. That keeps the extension lightweight and keeps the main model layer in one place.

**background work is helpful, not required for correctness**  
iOS decides when background refresh runs. SetWise refreshes the important state when the app launches, returns to the foreground, or workout data changes, so an unpredictable background schedule cannot break the core experience.

**finished workouts are history**  
Completed sessions are treated as snapshots of what actually happened. Program edits should not quietly rewrite the workout you completed last Tuesday.

---

## privacy by design

SetWise is deliberately boring about your data:

- no account
- no advertising
- no tracking pixels or in-app analytics SDK
- workout history stays on-device
- local export when you want a copy
- delete-all-data flow in Settings
- purchases handled by Apple
- widgets share only a lightweight local snapshot

Full policy: **[setwise.senithumesha.com/privacy](https://setwise.senithumesha.com/privacy)**

---

## about this repository

This is the **public product + engineering showcase** for SetWise.

The production application source is kept private. This repo exists so I can share the product, screenshots, architecture decisions, and the parts of the build I find interesting without publishing the commercial codebase or release configuration.

If you are here because of my GitHub profile, this is the native-iOS side quest that got a little out of hand.

---

<p align="center">
  <strong>SetWise: Workout & Recovery</strong><br />
  built by <a href="https://github.com/SenithUmesha">Senith Umesha</a>
</p>

<p align="center">
  <a href="https://apps.apple.com/us/app/setwise-workout-recovery/id6796000747">App Store</a>
  ·
  <a href="https://setwise.senithumesha.com/">setwise.senithumesha.com</a>
</p>
