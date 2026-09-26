# SetWise — engineering notes

This is the technical companion to the public [SetWise README](../README.md).

The production source lives in a private repository, so this document focuses on the architecture, data flow, release boundaries, and platform decisions behind the app rather than reproducing commercial implementation code.

> **Current release:** SetWise `2.0.0` build `33`, live on the App Store.

---

## 1. goals that shaped the architecture

SetWise is built around a few product constraints that are also architectural constraints:

- native iOS / iPadOS
- useful completely offline
- no account required
- no backend dependency for workout data
- no advertising or in-app analytics SDK
- unlimited core workout logging without a subscription
- deterministic training/recovery calculations that can be tested
- widgets and system integrations without duplicating the main persistence layer
- user-controlled import/export
- a real App Store release path including StoreKit, TestFlight, privacy configuration, and release builds

Those constraints push the app toward a deliberately simple rule:

> keep local training data authoritative, then derive everything else from it.

The architecture therefore favors clear domain boundaries over distributed infrastructure. SwiftData owns the primary product state; Apple platform integrations sit around that state as optional surfaces rather than becoming separate sources of truth.

---

## 2. SetWise 2.0 product loop

The original product centered heavily on recovery. Version 2.0 expands that into a complete strength-training loop:

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

That means the architecture now has to support more than a workout logger:

- saved plans and templates
- fixed-weekday and rotating schedules
- freestyle workouts
- active workout persistence
- multiple set types
- supersets and circuits
- exercise substitutions
- completed-session editing
- history and calendar views
- exercise progress
- muscle distribution / insights
- recovery context
- monthly reports
- yearly recaps
- import/export
- Free/Pro feature boundaries
- widgets, intents, notifications, and background refresh

The challenge is not simply adding these features. It is making them projections of the same training history rather than separate mini-products with incompatible data models.

---

## 3. app structure

At a high level the production project follows this shape:

```text
SetWise/
├── App/
├── Data/
├── DesignSystem/
├── Domain/
├── Features/
├── Intents/
├── Notifications/
├── Purchases/
├── Refresh/
├── Settings/
└── Shared/

SetWiseWidget/
SetWiseTests/
SetWiseUITests/
```

The dependency direction is intentionally straightforward:

```text
SwiftUI feature views
        ↓
feature state / view models
        ↓
use cases + domain services
        ↓
repositories
        ↓
SwiftData + lightweight Apple storage + system frameworks
```

There is no benefit in turning a product this size into an imitation of a huge enterprise backend. The useful boundary is the one that keeps product rules testable and prevents every screen from talking directly to persistence, StoreKit, notifications, or widget state.

---

## 4. persistence: local first because that is the product

Workout state should not depend on network reachability.

SetWise stores its primary user-created data locally with **SwiftData**. That includes the core training model: plans, exercises, active/completed sessions, set logs, history, progress inputs, and other durable workout state.

Simple preferences and lightweight integration state can live outside the main model graph when appropriate.

The important rule is:

> the device copy is not a cache of the "real" remote state — in 2.0, it is the real state.

That gives the product useful properties:

- opening History never requires a network fetch
- starting or resuming a workout does not wait for connectivity
- set edits become durable locally as the user trains
- completed workouts are immediately available to analytics
- recovery can be recalculated from the same training data
- exports can be generated without a server
- widgets can be produced from local state

Hosted exercise demonstration media is intentionally separate from private workout data. A guide may request reviewed media from the SetWise website, but workout history is not sent with that request.

---

## 5. plans, templates, and scheduling

SetWise 2.0 supports planned training without making the plan itself the historical truth.

Conceptually:

```text
plan / template
      ↓
scheduling rules
      ↓
next workout context
      ↓
materialized active session
```

Plans can evolve. Completed sessions should not.

That distinction lets a user change next week's program without silently mutating the workout they completed last week.

Scheduling includes both fixed-weekday and rotating approaches, while freestyle workouts bypass the plan completely and still land in the same completed-workout model.

---

## 6. active workout lifecycle

The active workout flow is where local persistence matters most.

A session can outlive the current view or foreground lifecycle. Users lock the phone, change music, answer messages, background the app, or return later.

So the active workout is treated as real product state rather than temporary SwiftUI state.

```text
plan / routine / freestyle
          ↓
start workout
          ↓
materialize active session
          ↓
edit exercises + sets
          ↓
persist while training
          ↓
finish workout
          ↓
freeze completed result into history
          ↓
rebuild derived training context
```

The workout model supports richer gym-floor behavior in 2.0, including previous-performance context, multiple set types, substitutions, supersets/circuits, rest utilities, and workout editing.

The UI can change significantly without changing the core rule: the finished session records what actually happened.

---

## 7. one training history, many projections

SetWise does not maintain unrelated databases for analytics, recovery, reports, and widgets.

Completed workouts form the canonical historical input for views such as:

- calendar presence
- exercise progress
- training volume
- muscle distribution
- recovery context
- monthly summaries
- comparisons
- yearly recaps
- widget snapshots

Conceptually:

```text
completed workouts
       ↓
domain projections
       ↓
progress / recovery / reports / recaps / widgets
```

Some derived data can be cached or precomputed for responsiveness, but the design goal is to keep those results reproducible from authoritative training history.

That reduces the chance that two screens disagree about the same workout.

---

## 8. recovery engine

Recovery in SetWise is deliberately **deterministic**.

It is not a machine-learning system and it is not presented as medical guidance.

The engine works from source information such as:

```text
completed training
elapsed time
muscle stimulus / training load
recent workout context
user-entered inputs where applicable
```

The central design choice is to avoid storing a calculated recovery percentage as permanent truth.

```text
source data + current time
          ↓
   RecoveryEngine
          ↓
 current recovery context
```

If the same source data is evaluated later, the result may change because time itself is an input.

That makes the engine explainable and unit-testable: given known history and a fixed clock, the output should be predictable.

Recovery wording is intentionally framed as training context. The app avoids presenting readiness calculations as diagnosis, injury detection, or medical certainty.

---

## 9. progress, reports, and recaps

The deeper review layer in 2.0 is still based on completed training rather than manually maintained analytics state.

The product can derive:

- exercise trends
- volume and performance views
- muscle distribution
- monthly reports
- period comparisons
- yearly recap data

The important architectural property is consistency. A report and an exercise-progress screen may present different views of the data, but they should start from the same completed sessions and domain rules.

This is also where the Free/Pro boundary is enforced as a product-access concern rather than by creating separate storage models for paying users.

---

## 10. Free / Pro boundary

SetWise Pro is a **single lifetime StoreKit 2 purchase** with restore support.

The free product keeps the core workout loop useful, including unlimited workout logging and raw history. Pro expands depth and range: more planning capacity, longer analysis windows, deeper exercise/muscle insights, older reports, recaps, widgets, and advanced customization.

The purchase layer is kept separate from feature implementation so product code can ask a capability question such as:

```text
is this feature available for the current entitlement?
```

instead of embedding StoreKit transaction handling throughout SwiftUI views.

Localized pricing is displayed from StoreKit at runtime. The application should not treat a hard-coded currency string as the source of truth for the purchase price.

---

## 11. import and export

Version 2.0 expands data portability while keeping the app local-first.

Import/export flows include user-controlled mechanisms such as:

- Hevy CSV import processed on-device
- JSON export
- local recovery / portability paths
- delete-all-data controls

Imported records must normalize into SetWise's own stable domain model rather than making external identifiers or source formats authoritative forever.

The same principle applies to exports: exporting data should not require a SetWise account or remote service.

---

## 12. exercise library and media

SetWise owns stable exercise identifiers and curated exercise metadata.

Hosted animation/media is treated as reviewed content, not an automatic lookup where a vaguely similar movement is considered good enough.

The production workflow therefore separates:

```text
exercise identity + metadata
          ↓
reviewed media mapping
          ↓
allowlisted hosted asset
```

If no reviewed animation exists, the product can show a placeholder rather than silently substituting a different exercise.

That keeps the training model stable even as media coverage grows independently.

---

## 13. WidgetKit without sharing the whole database

The widget extension does **not** own the full application model layer.

Instead, the main app writes a small Codable snapshot into an **App Group** container. WidgetKit reads that snapshot and renders from it.

```text
SwiftData + domain logic
         ↓
main app calculates current state
         ↓
WidgetSnapshot
         ↓
App Group container
         ↓
WidgetKit timeline
```

The snapshot can include things such as:

- generated-at timestamp
- recovery summary
- next-workout context
- streak/calendar summary
- values required by supported widget families

A widget snapshot is derived state. It is not a second source of truth.

If the snapshot is absent or stale, the extension should degrade safely instead of trying to recreate the entire application model in a second process.

---

## 14. background refresh

SetWise uses background refresh to improve freshness, not to guarantee correctness.

That distinction matters because **iOS owns the schedule**.

The application requests background execution, but core correctness cannot depend on a task running at a specific time. Important state is also refreshed during deterministic lifecycle events such as:

- app launch
- returning to foreground
- completing or editing a workout
- changing relevant settings

Background work can then improve supporting surfaces by refreshing snapshots, reloading widget timelines, updating reminder schedules, and preparing the next refresh request.

If iOS delays background work, opening SetWise must still produce correct state from local source data.

---

## 15. notifications and App Intents

SetWise uses **local notifications** rather than push infrastructure for product reminders that can be determined entirely on-device.

Notification handling still has normal platform concerns:

- permission state
- user preferences
- scheduling/replacing pending requests
- notification actions
- deep links into relevant flows

Selected capabilities are also exposed through **App Intents** so Shortcuts and system surfaces can enter the same underlying product services.

The important boundary is that an intent should call the same application/domain logic as the UI rather than reimplementing product rules in a parallel code path.

---

## 16. privacy posture

Privacy is an architectural choice here, not only App Store copy.

SetWise 2.0 is designed around:

```text
no SetWise account
no workout-data backend
no advertising SDK
no in-app analytics / tracking SDK
local SwiftData persistence
user-controlled import/export
local deletion
Apple-managed purchase processing
```

The project includes an Apple privacy manifest and public privacy/support documentation.

Hosted exercise media and public website/support resources are normal network-facing content paths; private workout history and plans are not sent along with those requests.

Public privacy policy: [setwise.senithumesha.com/privacy](https://setwise.senithumesha.com/privacy)

---

## 17. iCloud is intentionally post-2.0

Private iCloud / CloudKit sync is planned, but it is **not included in SetWise 2.0.0 build 33**.

That is an intentional release boundary.

Introducing synchronization changes the meaning of authority, conflict handling, migration, deletion, recovery, and multi-device behavior. It deserves a separately tested migration rather than being added late to an already finalized major release.

The intended direction is to preserve local-first behavior while using the user's private CloudKit database for durable cross-device sync, with independent export/recovery remaining available.

Until that work ships, the local SwiftData store remains authoritative.

---

## 18. testing

The production project contains both **unit tests** and **UI tests**.

High-value unit-test targets are the pieces that should remain deterministic:

- recovery calculations
- scheduling rules
- workout-selection logic
- history/progress projections
- report calculations
- data compatibility and migrations
- entitlement-access rules where isolated from StoreKit
- import normalization

UI coverage focuses on release-critical flows such as:

- starting planned and freestyle workouts
- editing set rows
- substitutions / workout editing
- resuming an active session
- completing a workout
- navigating plans, history, and analytics
- purchase/restore surfaces
- import and migration flows

The goal is not an arbitrary coverage percentage. It is making the core workout loop and product rules expensive to break accidentally.

---

## 19. release engineering

SetWise goes through the full native App Store path rather than stopping at a simulator demo.

That includes:

- app + widget targets
- App Groups and entitlements
- signing and archive validation
- privacy manifest work
- App Store metadata and screenshots
- TestFlight validation
- StoreKit product configuration
- support and privacy URLs
- release versioning
- shipped-build discipline

For the current major release:

| | |
|---|---|
| Version | `2.0.0` |
| Build | `33` |
| Status | Live on the App Store |
| Minimum OS | iOS 17 |
| iCloud sync | Not included in this binary |

Once a build ships, it is treated as frozen. New feature work moves to the next build/version rather than changing the meaning of a production binary after release.

The public App Store listing is here:

**[SetWise: Workout & Recovery](https://apps.apple.com/us/app/setwise-workout-recovery/id6796000747)**

---

## 20. decisions i would keep

If I rebuilt the product today, I would keep these choices:

1. **local source of truth** — it matches the gym-floor product better than introducing remote dependency for its own sake.
2. **completed workouts as historical snapshots** — future plan edits should not rewrite the past.
3. **deterministic recovery** — explainable and testable beats fake intelligence here.
4. **analytics as projections of completed training** — one history should power many views.
5. **snapshot-based widgets** — much cleaner than making an extension own the entire application database.
6. **background refresh as an optimization** — never depend on an OS-controlled schedule for correctness.
7. **StoreKit behind capability boundaries** — purchase state should not leak into every screen.
8. **Free as a real product** — paid depth is healthier than turning core logging into a subscription gate.
9. **sync as a deliberate migration** — multi-device state deserves its own design and testing cycle.
10. **native system features as product surfaces** — widgets, intents, notifications, and StoreKit are part of the app, not demo checkboxes.

---

## links

- [Product README](../README.md)
- [SetWise website](https://setwise.senithumesha.com/)
- [App Store](https://apps.apple.com/us/app/setwise-workout-recovery/id6796000747)
- [Privacy](https://setwise.senithumesha.com/privacy)
- [Support](https://setwise.senithumesha.com/support)
