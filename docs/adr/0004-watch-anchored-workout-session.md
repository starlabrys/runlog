# The Apple Watch anchors the workout session (Phase 2)

In Phase 2 the watchOS app owns the `HKWorkoutSession`: it keeps the workout alive in the background and reads heart rate directly. The iPhone handles GPS and the main UI. The two mirror each other through the workout session, and opening the app on the phone launches the watch app via `HKHealthStore.startWatchApp(with:)`.

## Why

A watchOS `HKWorkoutSession` gets reliable background execution and first-class heart-rate access. A phone-anchored session hits more iOS background-execution limits and has no direct heart-rate source. Since Phase 1 is already phone-only, a reader might assume Phase 2 stays phone-anchored; it does not.

## Consequences

- Heart rate is streamed watch -> phone live, and gaps are reconciled from HealthKit when the Session ends.
- If the phone's battery dies mid-run, the watch continues on a best-effort basis.
