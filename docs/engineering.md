# SetWise — engineering notes

This is the technical companion to the public [SetWise README](../README.md).

The production source lives in a private repository, so this document focuses on architecture, data flow, release boundaries, and platform decisions rather than reproducing commercial implementation code.

> **Current release state:** SetWise `2.1.0` build `34` was submitted for App Review on October 13, 2026. `2.0.0 (33)` remains the live App Store version until Apple approves/releases the update.

---

## 1. product constraints that shape the architecture

SetWise is designed around a few rules that are also engineering constraints:

- native iOS / iPadOS
- useful completely offline
- no SetWise account required
- no custom backend dependency for workout data
- no advertising or in-app analytics SDK
- unlimited core workout logging without a subscription
- deterministic training/recovery calculations
- widgets and system integrations without duplicating the main persistence layer
- private iCloud sync without making the gym-floor workflow network-dependent
- independent backup/restore because synchronization is not disaster recovery
- a real App Store path including StoreKit, CloudKit, TestFlight, privacy configuration, and signed release builds

The central rule is:

> make local training state durable first, then synchronize and derive external surfaces from it.

---

## 2. the training loop

SetWise is built around one connected strength-training loop:

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

That single model supports saved plans, rotating/fixed schedules, freestyle workouts, active workout persistence, multiple set types, supersets/circuits, substitutions, editable completed sessions, History, progress, recovery context, reports, recaps, imports, widgets, and Free/Pro boundaries.

The important design property is that these are projections of the same training history instead of separate mini-products with incompatible sources of truth.

---

## 3. project shape

The private production project is organized broadly as:

```text
SetWise/
├── App/
├── Data/
│   ├── Backup/
│   ├── Cloud/
│   ├── Repositories/
│   └── SwiftData/
├── DesignSystem/
├── Domain/
├── Features/
├── Intents/
├── Notifications/
├── Purchases/
├── Refresh/
└── Shared/

SetWiseWidget/
SetWiseTests/
SetWiseUITests/
```

Dependency direction stays straightforward:

```text
SwiftUI feature views
        ↓
feature state / view models
        ↓
use cases + domain services
        ↓
repositories
        ↓
local SwiftData store + Apple platform integrations
```

CloudKit, StoreKit, notifications, widget snapshots, and file import/export live around the product model rather than leaking into every screen.

---

## 4. local-first persistence

Workout state should never depend on network reachability.

SetWise stores its primary user-created state locally with SwiftData. That includes durable exercise definitions, workout programs, completed sessions, recovery check-ins, and active workout sessions.

The local store gives the app the behavior a gym-floor product needs:

- History opens without a network fetch
- starting/resuming a workout does not wait for iCloud
- set edits become durable locally while training
- a completed workout is available to analytics immediately
- recovery/progress can be derived from the same local history
- widgets can be regenerated from local state
- exports can be generated without a remote service

The physical 2.1 release-candidate test completed a workout with the phone genuinely offline, relaunched the target app while still offline, retained the exact workout once, then synchronized that workout after reconnect.

---

## 5. private CloudKit sync in 2.1

Version 2.1 adds synchronization through the user's **private CloudKit database**.

Conceptually:

```text
local SwiftData store
       ⇅
private CloudKit database
       ⇅
other eligible Apple devices
```

The current durable synchronized model set covers:

- workout sessions
- workout programs
- exercise definitions
- recovery check-ins
- active workout sessions

The application does not run a custom SetWise training-data backend and does not create a SetWise account.

Cloud availability is also not presented as a fake "everything is synced" indicator. Account/container availability and confirmed record propagation are different facts.

### conflict behavior

A particularly important 2.1 edge case was concurrent active workouts on two endpoints.

Instead of collapsing two distinct active drafts into one UI state, the app preserves both logical IDs and presents a deterministic chooser. Selection/discard operate on the exact UUID; finishing one draft removes only that draft and produces one completed workout.

That turns a synchronization collision into an explicit product state instead of silent data loss.

---

## 6. sync is not backup

Cloud synchronization is intentionally separate from backup.

A synchronized edit or deletion can propagate, so SetWise 2.1 adds a versioned JSON recovery format that is user-controlled and independent of CloudKit.

The recovery flow supports:

- local export
- restore preview
- Merge
- Replace
- idempotent repeated merge behavior
- compatibility handling for earlier backup versions where supported

The verified physical 2.1 backup contained 12 workouts, 2 recovery check-ins, 2 programs, and 153 exercises. Restoring that exact file into an independent non-CloudKit store preserved logical IDs, re-exported byte-identically at the original timestamp, and inserted zero durable records on a second merge.

Active unfinished workout drafts remain excluded from the disaster-recovery backup by design.

---

## 7. safe migration and failure handling

Introducing CloudKit changes persistence constraints, so 2.1 was treated as a migration rather than a late toggle.

The release work included:

- a synthetic populated 2.0 store fixture
- CloudKit-compatible defaults for mirrored nonoptional fields
- stable logical IDs across migration
- local store preservation checks
- explicit handling when a persistent store cannot be opened
- no silent fallback to a writable ephemeral store for real workout data

The physical TestFlight upgrade preserved the populated 2.0 dataset, and a later same-device uninstall/reinstall allowed Production CloudKit to reconstruct the durable workout history naturally rather than injecting the old SQLite file.

---

## 8. plans, active workouts, and historical snapshots

Plans can evolve. Completed sessions should not.

```text
plan / template
      ↓
scheduling rules
      ↓
next-workout context
      ↓
materialized active session
      ↓
completed historical snapshot
```

A user can change next week's program without rewriting last week's completed workout.

Active sessions are persisted product state rather than temporary SwiftUI state because users lock the phone, change music, answer messages, background the app, lose connectivity, or resume later.

---

## 9. one history, many projections

Completed workouts are the canonical historical input for:

- raw History
- calendar presence
- exercise progress
- training volume
- muscle distribution
- recovery context
- monthly summaries
- comparisons
- yearly recaps
- widget snapshots

```text
completed workouts
       ↓
domain projections
       ↓
progress / recovery / reports / recaps / widgets
```

Some values may be cached for responsiveness, but the design goal is reproducibility from authoritative training history.

---

## 10. deterministic recovery

Recovery in SetWise is deterministic rather than a black-box coaching system.

Inputs include completed training, elapsed time, muscle stimulus/load, recent workout context, and user-entered signals where applicable.

```text
source data + current time
          ↓
   RecoveryEngine
          ↓
 current recovery context
```

The app avoids presenting readiness as diagnosis, injury detection, or medical certainty.

---

## 11. StoreKit and the Free / Pro boundary

SetWise Pro is a **single lifetime StoreKit 2 purchase** with restore support.

Free keeps the core workout loop useful, including unlimited workout logging and raw history. Pro expands planning capacity, longer analysis ranges, deeper exercise/muscle insights, older reports, recaps, widgets, and advanced customization.

The purchase layer exposes capabilities rather than scattering StoreKit transaction code throughout feature views.

Localized pricing comes from StoreKit `displayPrice` at runtime. The public US `$9.99` value is only a storefront reference, not a hard-coded in-app price source.

---

## 12. exercise media as reviewed content

SetWise owns stable exercise identifiers and curated metadata.

Hosted media follows:

```text
exercise identity + metadata
          ↓
reviewed movement mapping
          ↓
provenance + digest validation
          ↓
allowlisted hosted GIF
```

Current catalog state:

- 149 built-in exercises
- 120 reviewed hosted animations
- 29 intentional placeholders
- 26 reviewed animations added in the 2.1 expansion

If an exact reviewed animation does not exist, the UI prefers a placeholder over a visually similar but incorrect movement.

---

## 13. widgets without sharing the whole database

The widget extension does not own the full application model graph or CloudKit configuration.

Instead:

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

The snapshot can include readiness, next-workout context, streak/calendar summaries, and values used by the supported widget families.

A widget snapshot is derived state, not another source of truth. The 2.1 physical release smoke directly inspected rendered Home Screen widgets rather than relying only on App Group files.

---

## 14. background work and remote changes

Background refresh improves freshness; it does not define correctness because iOS owns scheduling.

Important refresh work also happens on deterministic lifecycle events such as launch, foregrounding, workout completion/editing, and relevant settings changes.

The 2.1 app also has the distribution entitlements needed for CloudKit remote-change handling, while widget data still flows through the local App Group snapshot.

A documented verification limitation remains: a second physical iPhone was unavailable, so real-device-to-real-device APNs timing/background widget behavior was not directly exercised. Development cross-endpoint convergence used a physical iPhone plus signed-in simulator, while Production/TestFlight behavior was verified on the physical phone.

---

## 15. privacy posture

Privacy is an architectural choice rather than only marketing copy.

SetWise is designed around:

```text
no SetWise account
no custom workout-data backend
no advertising SDK
no in-app analytics / tracking SDK
local-first SwiftData persistence
private CloudKit sync in enabled builds
independent user-controlled backup/export
local deletion controls
Apple-managed purchase processing
```

Hosted exercise media and the public website/support pages are network-facing content paths, but exercise-media delivery does not include private workout/program/recovery records.

Public privacy policy: [setwise.senithumesha.com/privacy](https://setwise.senithumesha.com/privacy)

---

## 16. testing and release verification

The production project contains both unit tests and UI tests, plus physical-device release checks for flows where simulator-only coverage is not enough.

For the verified 2.1 build-34 release candidate:

- **261/261 unit tests passed**
- **60/60 UI tests passed**
- Debug and Release simulator builds passed
- generic signed-device build passed
- static analysis passed
- signed archive/signature checks passed
- app/widget entitlement shapes were inspected
- populated 2.0 → 2.1 physical upgrade passed
- Production CloudKit records were confirmed
- genuine physical offline workout + offline target restart passed
- same-device TestFlight reinstall/reconstruction passed
- physical backup/restore preview and independent restore passed
- Light/Dark, exercise media, and rendered widget smoke passed

The release process deliberately distinguishes simulator evidence, physical-device evidence, Production CloudKit evidence, and inference rather than calling all of them the same thing.

---

## 17. release engineering

SetWise uses the full native App Store path:

- app + widget targets
- App Groups and CloudKit entitlements
- signing/archive validation
- privacy manifest and public privacy/support pages
- App Store metadata/screenshots
- TestFlight validation
- StoreKit configuration
- Production CloudKit schema promotion/inspection
- monotonically increasing build numbers
- shipped-build discipline

Current state:

| | |
|---|---|
| Submitted version | `2.1.0` |
| Submitted build | `34` |
| App Review state | Submitted October 13, 2026; awaiting Apple review |
| Live App Store version | `2.0.0 (33)` until 2.1 is released |
| Minimum OS | iOS 17 |
| Private iCloud sync | Included in 2.1 |
| Independent backup/restore | Included in 2.1 |

Build 34 remains the verified binary because later release-candidate commits changed documentation/UI-test code rather than shipped runtime behavior. No unnecessary build 35 was created.

---

## 18. decisions i would keep

1. **local-first workout behavior** — network reachability should not be a gym-floor dependency.
2. **private CloudKit instead of a custom account backend** — use the user's Apple infrastructure when sync is needed.
3. **sync and backup as different systems** — synchronization cannot be the only recovery strategy.
4. **completed workouts as historical snapshots** — future plan edits should not rewrite the past.
5. **explicit active-draft collision handling** — preserve IDs instead of silently picking a winner in the UI.
6. **deterministic recovery** — explainable and testable beats fake intelligence.
7. **analytics as projections of completed training** — one history powers many views.
8. **snapshot-based widgets** — much cleaner than making an extension own the entire model graph.
9. **background refresh as an optimization** — correctness must survive delayed execution.
10. **StoreKit behind capability boundaries** — purchase state should not leak into every feature.
11. **reviewed media with placeholders** — correctness and provenance beat maximum coverage.
12. **physical release verification for persistence changes** — migrations and offline behavior deserve more than simulator confidence.

---

## links

- [Product README](../README.md)
- [SetWise website](https://setwise.senithumesha.com/)
- [App Store](https://apps.apple.com/us/app/setwise-workout-recovery/id6796000747)
- [Privacy](https://setwise.senithumesha.com/privacy)
- [Support](https://setwise.senithumesha.com/support)
