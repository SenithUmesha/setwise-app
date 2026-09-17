# SetWise — engineering notes

This is the technical companion to the public [SetWise README](../README.md).

The production source lives in a private repository, so this document focuses on the architecture, data flow, and platform decisions behind the shipped app rather than reproducing implementation code.

---

## 1. goals that shaped the architecture

SetWise had a few constraints from the start:

- native iOS first
- useful completely offline
- no account required
- no backend for workout data
- no analytics SDK
- widget support without coupling the widget to the main app's persistence layer
- deterministic recovery logic that can be tested
- a real App Store release path, including purchases, privacy configuration, TestFlight, and release builds

Those constraints pushed the app toward a fairly simple structure: keep the local store authoritative, isolate domain logic from SwiftUI, use Apple frameworks directly, and make system integrations optional layers around the core product.

---

## 2. app structure

At a high level the production project is organized like this:

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

The dependency direction is intentionally boring:

```text
SwiftUI feature views
        ↓
feature state / view models
        ↓
domain services + use cases
        ↓
repositories
        ↓
SwiftData / UserDefaults / Apple system APIs
```

There is no reason for a project this size to imitate a giant enterprise architecture. The goal is to keep boundaries clear enough that product logic can be tested without turning every screen into five layers of ceremony.

---

## 3. persistence: local first, because that is the product

Workout state should not depend on network reachability.

SetWise stores its primary user-created data locally with **SwiftData**. That includes the things the product actually cares about: plans, exercises, active/completed sessions, set logs, history, and recovery inputs/context.

Simple preferences remain outside the main model graph when they do not need to be there.

The important rule is:

> the device copy is not a cache of the "real" remote state — it is the real state.

That changes a lot of product behaviour for the better:

- opening History never needs a fetch
- starting or resuming a workout does not wait for a connection
- finishing a set is immediately durable locally
- recovery can be recalculated from the same data the UI already owns
- exports can be generated without a server

SetWise does request hosted exercise demonstration media when a user opens a guide and media is available. That content path is intentionally separate from workout data.

---

## 4. active workout lifecycle

The active workout flow is one of the places where local persistence matters most.

A workout can outlive the current screen or app foreground session. Users lock the phone, answer messages, switch music, accidentally background the app, or return later.

So the active session is treated as real state rather than temporary view state.

```text
program / routine
      ↓
start workout
      ↓
materialize active session
      ↓
edit sets / reps / load / RIR
      ↓
persist as the session changes
      ↓
finish workout
      ↓
freeze completed result into history
      ↓
update recovery + progress context
```

A key product decision is that a completed workout should describe what actually happened at that time. Editing the underlying plan later must not silently rewrite old history.

---

## 5. recovery engine

Recovery in SetWise is deliberately **deterministic**.

It is not a machine-learning model and it is not presented as medical guidance.

The engine works from source information such as:

```text
completed training
elapsed time
muscle stimulus / training load
recent workout context
user-entered values where applicable
```

The central design choice is to avoid storing a calculated recovery percentage as permanent truth.

Conceptually:

```text
source data + current time
          ↓
   RecoveryEngine
          ↓
 current recovery view
```

If the same source data is evaluated six hours later, the result may be different because time itself is an input.

A lightweight result can be cached for presentation layers such as widgets, but the app's durable truth remains the underlying training data.

This also makes the engine straightforward to unit test: given a known workout history and a fixed clock, the result should be predictable.

---

## 6. progress and review data

SetWise derives review views from completed training rather than maintaining a second manually-updated analytics store.

The same session history powers:

- calendar presence
- exercise progress
- training volume views
- muscle distribution
- monthly summaries
- yearly recap data
- recovery context

That keeps analytics closer to a projection of source history instead of another database that can drift out of sync.

Where calculations are more expensive, derived results can be prepared for the UI, but completed sessions remain the canonical input.

---

## 7. WidgetKit without sharing the whole database

The widget extension does **not** own the full application model layer.

Instead, the app writes a small Codable snapshot into an **App Group** container. The widget reads that snapshot and renders from it.

```text
SwiftData + domain logic
         ↓
main app calculates current state
         ↓
WidgetSnapshot
         ↓
App Group shared container
         ↓
WidgetKit timeline
```

This gives the extension only what it needs and avoids pushing persistence complexity into two processes.

The snapshot can include things like:

- generated-at timestamp
- recovery summary
- next workout context
- calendar/streak summary
- values needed by supported widget families

If a snapshot is missing or old, the widget can fail gracefully rather than becoming another source of truth.

---

## 8. background refresh

SetWise uses background refresh to improve freshness, not to guarantee correctness.

That distinction matters because **iOS owns the schedule**.

The app can request background execution, but it cannot assume a job will run at an exact time. Therefore the core product recalculates important state during deterministic lifecycle events too, such as:

- app launch
- returning to foreground
- completing or editing a workout
- relevant settings changes

Background refresh can then:

- refresh shared widget state
- reload widget timelines
- update local reminder scheduling
- prepare the next refresh request

If iOS delays that work, opening SetWise still produces correct state from local source data.

---

## 9. notifications

SetWise uses **local notifications** rather than push infrastructure.

That fits the rest of the architecture: reminders are based on information already available on the device, so a remote notification service would add infrastructure without improving the core use case.

Notification handling includes the usual platform concerns:

- permission state
- user-configured preferences
- scheduling / replacing pending requests
- notification actions
- deep links back into the relevant app flow

---

## 10. App Intents and system integration

The app exposes selected actions through **App Intents** so useful product behaviour can exist outside the main app UI.

The important architectural bit is that intents call into the same underlying domain/application services instead of reimplementing logic specifically for Shortcuts.

That keeps system features as another entry point into the product rather than a separate mini-app.

---

## 11. purchases

SetWise Pro is implemented as a **lifetime in-app purchase** with a restore path.

The purchase layer is kept separate from the feature code so the app can ask a simple product question such as:

```text
is this capability available?
```

instead of scattering transaction logic through screens.

The free app remains usable without the purchase. Pro unlocks deeper history/review capabilities and widgets rather than turning the basic workout logger into a trial shell.

Purchase processing is handled by Apple; SetWise does not store payment details.

---

## 12. privacy posture

Privacy is an architectural choice here, not just App Store copy.

The production app is designed around:

```text
no account
no workout-data backend
no advertising SDK
no in-app analytics / tracking SDK
local persistence
local export
local deletion
Apple-managed purchase processing
```

The project also includes an Apple privacy manifest and a hosted public privacy policy.

The only normal network-facing product content is things such as hosted exercise guide media and public website/support resources. Workout history and program data are not sent along with those media requests.

Public privacy policy: [setwise.senithumesha.com/privacy](https://setwise.senithumesha.com/privacy)

---

## 13. testing

The production project contains both **unit tests** and **UI tests**.

The most valuable unit-test targets are the parts that should stay deterministic:

- recovery calculations
- recommendation / workout-selection rules
- schedule behaviour
- history / progress projections
- data compatibility and migrations
- purchase-access rules where they can be isolated from StoreKit

UI coverage focuses on high-risk flows such as:

- starting a workout
- editing set rows
- resuming an active workout
- completing a workout
- navigating plans/history
- import or migration flows where applicable
- release-critical screens

The point is not to chase an arbitrary coverage percentage. It is to make the product rules and the main workout loop expensive to accidentally break.

---

## 14. release engineering

SetWise went through the full native-iOS release path rather than stopping at a simulator demo.

That includes:

- separate app + widget targets
- App Groups and entitlements
- signing and archive validation
- privacy manifest work
- App Store metadata and screenshots
- TestFlight / App Store build handling
- in-app purchase configuration
- support + privacy URLs
- real release versioning

The public App Store listing is here:

**[SetWise: Workout & Recovery](https://apps.apple.com/us/app/setwise-workout-recovery/id6796000747)**

---

## 15. what i would keep the same

If I rebuilt the app today, I would keep these decisions:

1. **local source of truth** — it matches the actual product better than introducing sync for its own sake.
2. **deterministic recovery** — explainable and testable beats fake intelligence here.
3. **snapshot-based widgets** — much cleaner than making an extension understand the entire app store.
4. **background refresh as an optimisation** — never depend on an OS-controlled schedule for core correctness.
5. **native system features as first-class product surfaces** — widgets, intents, notifications, and StoreKit are part of the app, not demo checkboxes.

---

## links

- [Product README](../README.md)
- [SetWise website](https://setwise.senithumesha.com/)
- [App Store](https://apps.apple.com/us/app/setwise-workout-recovery/id6796000747)
- [Privacy](https://setwise.senithumesha.com/privacy)
- [Support](https://setwise.senithumesha.com/support)
