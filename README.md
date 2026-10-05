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

SetWise 2.0 expands the original recovery-focused app into a complete training loop:

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

It brings workout planning, freestyle training, richer set logging, editable history, progress analytics, muscle insights, reports, yearly recaps, import/export, recovery, widgets, and a Free/Pro model into one native iOS product.

---

## why i built it

Most workout apps are good at recording sets. I wanted the context around those sets too: what was planned, what actually happened, what changed, and what should be visible before the next session.

So SetWise is built around completed training rather than around a social feed or a cloud account.

- no account required to train
- no ads
- no recurring subscription
- no in-app analytics SDK
- no AI coach pretending to know your body
- core workout logging works offline

---

## SetWise 2.0 in screenshots

<table>
  <tr>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/1.png" width="210" alt="SetWise 2.0 screenshot 1" /></td>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/2.png" width="210" alt="SetWise 2.0 screenshot 2" /></td>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/3.png" width="210" alt="SetWise 2.0 screenshot 3" /></td>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/4.png" width="210" alt="SetWise 2.0 screenshot 4" /></td>
  </tr>
  <tr>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/5.png" width="210" alt="SetWise 2.0 screenshot 5" /></td>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/6.png" width="210" alt="SetWise 2.0 screenshot 6" /></td>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/7.png" width="210" alt="SetWise 2.0 screenshot 7" /></td>
    <td></td>
  </tr>
</table>

These are the same current screenshots used by my portfolio so the public product repo and portfolio stay aligned.

---

## plan

Build reusable training around the way you actually work out, or skip the plan and start freestyle.

- saved workout plans and templates
- fixed-weekday and rotating scheduling
- exercise prescriptions with target sets and rep ranges
- one-tap access to the next planned session
- freestyle workouts
- reusable routines that can evolve without rewriting completed history

## train

The active workout is designed for the few seconds between sets.

- previous-performance context beside the current work
- warm-up, working, back-off, drop, and failure set types
- flexible load, reps, and optional effort tracking
- supersets and circuits
- exercise substitution without rebuilding the session
- automatic rest timing with quick adjustments
- plate calculator and training utilities
- active-session persistence across app backgrounding and interruptions

### reviewed exercise media

The exercise library now has a larger reviewed media catalog. The production app only allowlists exercise animations after movement-match, provenance, digest, and visual review.

That means SetWise prefers an honest placeholder over silently showing a merely similar exercise. The current reviewed expansion includes **26 deployed and verified exercise GIFs**, aligned with the app catalog after the 2.0 release.

## finish + review

A completed workout becomes the historical snapshot of what actually happened. That same history powers:

- unlimited raw workout history
- a rolling 12-month calendar
- editable completed sessions
- exercise progress trends
- muscle distribution and training insights
- workout-derived recovery context
- monthly reports and comparisons
- yearly training recaps

Recovery is a training signal, not a medical claim. It provides context from logged training rather than diagnosing injury or guaranteeing readiness.

---

## useful before you pay

The free app keeps the core training loop genuinely useful.

It includes unlimited workout logging and raw workout history, core recovery context, exercise guides and utilities, local export, a rolling 12-month calendar, limited planning tools, and useful recent progress views.

**SetWise Pro** is one lifetime purchase. It expands planning capacity, longer progress ranges, exercise and muscle insights, older reports, yearly recaps, widgets, and advanced customization.

No recurring billing.

---

## import, export, and ownership

SetWise 2.0 supports user-controlled data movement without turning the app into a cloud-account product.

- on-device Hevy CSV import
- local user-controlled export
- JSON export/recovery paths
- delete-all-data controls
- no SetWise account required

The production app still treats the local SwiftData store as the source of truth.

Private iCloud / CloudKit sync is planned for a separately tested post-2.0 release. It is **not** part of build 33.

---

## architecture

```text
SwiftUI
   ↓
feature state + view models
   ↓
use cases + domain services
   ↓
repositories
   ↓
SwiftData + UserDefaults / AppStorage + Apple system frameworks
```

Platform work includes:

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

The main app remains the source of truth. Widgets receive lightweight App Group snapshots and degrade safely if that snapshot is missing or stale.

More implementation notes live in **[docs/engineering.md](docs/engineering.md)**.

---

## privacy by design

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

The production application source is private. This repo exists so I can share the product, screenshots, architecture decisions, release thinking, and interesting engineering work without publishing commercial source, signing configuration, or release infrastructure.

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
