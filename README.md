<p align="center">
  <img src="https://setwise.senithumesha.com/images/setwise/app-icon.png" width="112" alt="SetWise app icon" />
</p>

<h1 align="center">SetWise</h1>

<p align="center">
  <strong>Train with a plan. Learn from every session.</strong>
</p>

<p align="center">
  A private, local-first iPhone & iPad strength-training app that connects planning, set-by-set logging, recovery, progress, and review.
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
  <code>SwiftUI</code> · <code>SwiftData</code> · <code>WidgetKit</code> · <code>App Intents</code> · <code>StoreKit 2</code> · <code>local-first</code>
</p>

<p align="center">
  <a href="https://github.com/SenithUmesha/setwise-app/actions/workflows/docs-check.yml"><img src="https://github.com/SenithUmesha/setwise-app/actions/workflows/docs-check.yml/badge.svg" alt="Docs integrity" /></a>
</p>

---

## SetWise 2.0

**Version 2.0.0 · build 33** is live on the **App Store**.

It is the first major expansion beyond the original recovery-focused release and the current production version of SetWise.

```text
Plan
  ↓
Train
  ↓
Finish
  ↓
Review
  ↓
Use the context in the next session
```

2.0 brings workout planning, freestyle training, richer set logging, history editing, progress analytics, muscle insights, reports, recaps, import/export, recovery, widgets, and the Free/Pro model into one product loop.

---

## why i built it

Most workout apps are good at recording sets. What I wanted was the part around those sets too: _what was I supposed to train, what did I actually do, what changed, and what context should I have before the next session?_

So SetWise became a simple loop built around completed training:

```text
plan or start freestyle
        ↓
log the sets you actually complete
        ↓
finish the session
        ↓
update history + recovery + progress
        ↓
review what changed
        ↓
come back with context for the next workout
```

No social feed. No account required to log a workout. No ads. No recurring subscription. No AI coach pretending to know your body.

It is intentionally a **private, local-first native iOS product** built around the gym session itself.

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
    <td align="center"><sub>train</sub></td>
    <td align="center"><sub>review</sub></td>
  </tr>
</table>

### plan

Build reusable training around the way you actually work out, or skip the plan and start freestyle.

- saved workout plans and templates
- fixed-weekday and rotating scheduling
- exercise prescriptions with target sets and rep ranges
- one-tap access to the next planned session
- freestyle workouts when the day does not fit the plan
- reusable routines that can evolve without rewriting completed history

### train

The active workout is designed for the few seconds between sets, not for sitting at a desk filling out a form.

- previous-performance context beside the current work
- warm-up, working, back-off, drop, and failure set types
- flexible load, reps, and optional effort tracking
- supersets and circuits
- exercise substitution without rebuilding the session
- automatic rest timing with quick adjustments
- plate calculator and training utilities
- reviewed exercise-guide media when available
- active-session persistence across app backgrounding and interruptions

### finish

A completed workout becomes a snapshot of what actually happened.

That matters because the plan can change tomorrow without rewriting last Tuesday. Completed sets feed the same local history used by recovery, progress, muscle analysis, reports, recaps, widgets, and system actions.

### review

The point of logging is to make the next session better informed.

- unlimited raw workout history
- rolling 12-month workout calendar
- exercise progress trends
- muscle distribution and training insights
- workout-derived recovery context
- monthly reports and comparisons
- yearly training recaps
- editable completed-session history

Recovery is a **training signal**, not a medical claim. It is derived from the work you log and is there to provide context, not diagnose injury or guarantee readiness.

---

## useful before you pay

The free app keeps the core training loop genuinely useful.

It includes unlimited workout logging and raw workout history, core recovery context, exercise guides and utilities, local export, a rolling 12-month calendar, limited planning tools, and useful recent progress views.

**SetWise Pro** is one lifetime purchase. It expands the product with deeper planning capacity, longer progress ranges, exercise and muscle insights, older reports, yearly recaps, widgets, and advanced customization.

No recurring billing. The workout logger is not a subscription trial shell.

---

## import, export, and ownership

SetWise 2.0 adds user-controlled data movement without turning the app into a cloud account product.

- on-device Hevy CSV import
- local user-controlled export
- JSON export/recovery paths
- delete-all-data controls
- no SetWise account required

The production app still treats the local SwiftData store as the source of truth.

Private iCloud/CloudKit sync is planned for a separately tested post-2.0 release. It is **not** part of build 33.

---

## the fun iOS stuff

SetWise was also my excuse to build a product properly around Apple platform features instead of treating iOS as just another deployment target.

```text
SwiftUI               screens + interaction
SwiftData              local plans, workouts, history + training state
WidgetKit              recovery, calendar, streak + next-workout surfaces
App Intents            Shortcuts / system actions
App Groups             lightweight widget snapshot sharing
UserNotifications      local reminders + training notifications
BGTaskScheduler        opportunistic background refresh
StoreKit 2             lifetime Pro purchase + restore flow
XCTest / UI tests      domain + workout-flow verification
TestFlight             release validation
Privacy manifest       explicit platform privacy configuration
```

The production app has **no user account, no workout-data backend, no ads, and no in-app analytics SDK**. Training data stays on-device. Exercise demo media is fetched only when needed and remains separate from private workout history.

If you're interested in the implementation choices rather than the product tour, they are documented in **[docs/engineering.md](docs/engineering.md)**.

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
Plans, workouts, set history, settings, and training-derived state live on the device. The app does not need a server round-trip to tell you what you did five minutes ago.

**completed workouts are immutable history by default**  
A finished session describes what actually happened. Editing a template or future plan should not silently rewrite previous training.

**recovery is calculated, not stored as fake truth**  
Recovery is derived from training source data and current time. A percentage can change as time passes; it should not become permanent truth simply because it was once calculated at a particular moment.

**analytics project from the same history**  
Progress, muscle distribution, reports, recaps, calendar state, and recovery all originate from completed training rather than being maintained as unrelated copies of the same truth.

**widgets get a snapshot, not the whole app database**  
The main app writes a small App Group snapshot for the widget extension. That keeps the extension lightweight and leaves authoritative calculations in the main product.

**background work is helpful, not required for correctness**  
iOS decides when background refresh runs. SetWise refreshes important state during deterministic lifecycle events too, so an unpredictable background schedule cannot break the core experience.

**Free should be a real workout app**  
Unlimited logging and raw history remain useful without payment. Pro sells depth and range rather than access to the basic act of recording a workout.

---

## privacy by design

SetWise is deliberately boring about your data:

- no account
- no advertising
- no social feed
- no tracking pixels or in-app analytics SDK
- workout history stays on-device
- local import/export under user control
- delete-all-data flow in Settings
- purchases handled by Apple
- widgets share only a lightweight local snapshot
- no iCloud training-data sync in 2.0.0 build 33

Full policy: **[setwise.senithumesha.com/privacy](https://setwise.senithumesha.com/privacy)**

---

## release status

| | |
|---|---|
| Marketing version | `2.0.0` |
| Build | `33` |
| Status | Live on the App Store |
| Minimum OS | iOS 17 |
| Purchase model | Free core + one-time lifetime SetWise Pro |
| Primary data model | Local-first SwiftData |
| Cloud sync | Planned post-2.0; not included in build 33 |

The shipped 2.0 binary is treated as frozen. New product work belongs in the next release rather than changing the meaning of a production build after release.

---

## about this repository

This is the **public product + engineering showcase** for SetWise.

The production application source is kept private. This repo exists so I can share the product, screenshots, architecture decisions, release thinking, and the parts of the build I find interesting without publishing the commercial codebase, signing setup, or release configuration.

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
