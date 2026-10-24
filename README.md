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
  <code>SwiftUI</code> · <code>SwiftData</code> · <code>CloudKit</code> · <code>WidgetKit</code> · <code>StoreKit 2</code> · <code>local-first</code>
</p>

<p align="center">
  <a href="https://github.com/SenithUmesha/setwise-app/actions/workflows/docs-check.yml"><img src="https://github.com/SenithUmesha/setwise-app/actions/workflows/docs-check.yml/badge.svg" alt="Docs integrity" /></a>
</p>

---

## SetWise 2.1

**Version 2.1.0 · build 34** was released on the **App Store on October 13, 2026**. A 2.1.1 stabilization update is being prepared separately; proposed iOS 17.0 compatibility remains unverified until build and device testing.

SetWise 2.1 keeps the gym-floor workflow local-first while adding private sync and a separate recovery path:

- private CloudKit sync through the user's own iCloud account
- core workout logging that continues to work offline
- independent versioned JSON backup and restore with preview, Merge, and Replace flows
- safe migration from the existing 2.0 local store
- deterministic handling for multiple synchronized active-workout drafts
- expanded reviewed exercise-demo coverage
- the same one-time SetWise Pro model with no recurring subscription

The release candidate was verified on the actual TestFlight build with a populated upgrade, same-device reinstall/reconstruction, a genuinely offline completed workout and offline restart, Production CloudKit record confirmation, backup/restore round trip, Light/Dark UI, hosted-media failure/recovery, and rendered Home Screen widgets.

---

## why i built it

Most workout apps are good at recording sets. I wanted the context around those sets too: what was planned, what actually happened, what changed, and what should be visible before the next session.

So SetWise is built around completed training rather than around a social feed or a proprietary cloud account.

- no SetWise account required to train
- no ads
- no recurring subscription
- no in-app analytics SDK
- no custom workout-data backend
- core workout logging works offline

---

## current product screenshots

<table>
  <tr>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/1.png" width="210" alt="SetWise screenshot 1" /></td>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/2.png" width="210" alt="SetWise screenshot 2" /></td>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/3.png" width="210" alt="SetWise screenshot 3" /></td>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/4.png" width="210" alt="SetWise screenshot 4" /></td>
  </tr>
  <tr>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/5.png" width="210" alt="SetWise screenshot 5" /></td>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/6.png" width="210" alt="SetWise screenshot 6" /></td>
    <td align="center"><img src="https://senithumesha.com/images/projects/setwise/7.png" width="210" alt="SetWise screenshot 7" /></td>
    <td></td>
  </tr>
</table>

These captures show the current 2.x product UI and are shared with my portfolio so the public product repo and portfolio stay aligned.

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
- persistent active sessions that survive app/background interruptions
- synchronized-draft collision handling that preserves distinct active workouts instead of silently discarding one

## sync without giving up offline training

SetWise 2.1 uses the user's **private iCloud / CloudKit database** for durable cross-device synchronization.

The local store is still the immediate working state. A workout can start, change, finish, and survive an app restart without a network connection; CloudKit reconciliation happens when connectivity is available.

SetWise does not create its own login system or operate a custom workout-data backend.

Sync is intentionally **not** treated as backup. An edit or deletion can synchronize too, so 2.1 adds a versioned JSON backup/restore path that remains independent of CloudKit.

---

## reviewed exercise media

SetWise owns stable exercise identities and only allowlists hosted media after movement-match, provenance, digest, and visual review.

The current catalog contains:

- **149 built-in exercises**
- **120 reviewed hosted animations**
- **29 intentional placeholders** where no exact reviewed media is available
- **26 newly reviewed animations** added in the 2.1 media expansion

SetWise prefers an honest placeholder over silently showing a merely similar movement.

---

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

## import, backup, restore, and ownership

SetWise supports user-controlled data movement without turning the app into a proprietary cloud-account product.

- on-device Hevy CSV import
- private iCloud sync in 2.1-enabled builds
- independent versioned JSON backup/export
- restore preview with Merge and Replace paths
- delete-all-data controls
- no SetWise account required

A CloudKit account-availability state is not presented as proof that every recent record has already synchronized. Backup remains a separate user-controlled recovery mechanism.

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
local SwiftData store ⇄ private CloudKit sync
   ↓
App Group snapshots / Apple system integrations
```

Platform work includes:

```text
SwiftUI               screens + interaction
SwiftData              local plans, workouts, history + training state
CloudKit               private durable-data synchronization
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

The local app store remains the immediate source for product behavior. CloudKit synchronizes durable model data; widgets receive lightweight App Group snapshots and do not own the full model graph.

More implementation notes live in **[docs/engineering.md](docs/engineering.md)**.

---

## privacy by design

- no SetWise account
- no advertising
- no social feed
- no tracking pixels or in-app analytics SDK
- local-first workout logging
- private iCloud sync handled through Apple's CloudKit in enabled builds
- no custom SetWise workout-data backend
- independent user-controlled backup/export
- delete-all-data flow in Settings
- purchases handled by Apple
- widgets share only a lightweight local snapshot

Full policy: **[setwise.senithumesha.com/privacy](https://setwise.senithumesha.com/privacy)**

---

## release status

| | |
|---|---|
| Submitted marketing version | `2.1.0` |
| Submitted build | `34` |
| Status | Released on the App Store on October 13, 2026 |
| Live App Store version | `2.1.0 (34)` |
| Minimum OS | Live 2.1 requires iOS 26.5; proposed 2.1.1 iOS 17 target requires testing |
| Purchase model | Free core + one-time lifetime SetWise Pro |
| Primary data model | Local-first SwiftData |
| Cloud sync | Private CloudKit sync in 2.1 |
| Recovery path | Independent versioned JSON backup/restore |

Build 34 was verified and released. A separate 2.1.1 maintenance candidate is in progress; its build 35 must pass Xcode and device checks before submission.

---

## about this repository

This is the **public product + engineering showcase** for SetWise.

The production application source is private. This repo exists so I can share the product, screenshots, architecture decisions, release thinking, and interesting engineering work without publishing commercial source, signing configuration, user data, or release infrastructure.

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
