# Phased delivery: iPhone-only on a free certificate first

**Phase 1** is an iPhone-only app signed with a free personal certificate: GPS + CoreMotion classification, Segments, Route, distance, pace, Splits, local storage, GPX/JSON export. No Apple Watch, no heart rate, no HealthKit, no backend.

**Phase 2**, after a paid Apple Developer Program membership ($99/yr) is bought: the Apple Watch app, heart rate, writing to Apple Health, the lock-screen Live Activity, and backend sync.

## Why

HealthKit's capability requires the paid membership, and a free personal certificate's provisioning profile expires every 7 days. Rather than pay upfront, Phase 1 proves the core bet — that automatic Running / Walking / Idle classification is good enough to rely on — using only frameworks a free certificate allows (`CoreLocation`, `CoreMotion`). The membership is bought once Phase 1 is usable day to day.

## Consequences

- Phase 1 cannot use `HKWorkoutSession` / `HKLiveWorkoutBuilder`, so it needs its own classification and auto-pause logic (see [0003](0003-on-device-motion-classification.md)).
- A future reader seeing hand-rolled motion code instead of HealthKit should read this before "fixing" it.
- Weekly re-signing is an accepted Phase 1 cost.
