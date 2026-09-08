# Phase 1 spec: iPhone-only Session recording with automatic Running / Walking / Idle classification

## Problem Statement

I run, and I want the route and the numbers kept as my own data. Keep — the app I use today — is heavy with ads, social features I don't want, and offers no API to get my history out.

Beyond that, every running app I've used makes me announce that I'm about to run. I open the app, find the start button, wait out a 3-2-1 countdown, and only then move. When I forget, the first kilometre is gone. When I stop at a traffic light, my average pace is quietly ruined unless I remember to press pause and press it again when the light changes. The app makes me do bookkeeping while I am trying to run.

There is also a second failure I hit regularly: I finish a run, walk home, and forget to press stop. The Record then claims an hour of "running" that was really me buying groceries.

## Solution

runlog records a **Session** from the moment the app opens. There is no start button and no countdown. The app watches motion and location and decides for itself when I am running, walking, or standing still, cutting the Session into **Running / Walking / Idle Segments**. **Running time** — the headline number — is the sum of the Running Segments only, so a warm-up walk and a red light never inflate it, and I never have to press pause.

The only thing I must do is press stop. If I forget, the Session ends itself once it passes a distance or duration limit I control.

When the classifier is wrong — a slow jog read as walking — I tap **Force start** or **Force end** to correct the current Segment on the spot.

Everything is stored on the phone as the single source of truth, and any **Workout Record** can be exported as GPX or JSON.

Phase 1 is iPhone-only on a free certificate ([ADR-0002](adr/0002-phased-delivery-free-cert-first.md)): no Apple Watch, no heart rate, no HealthKit, no backend.

## User Stories

### Recording a Session

1. As a runner, I want the Session to begin when I open the app, so that I never lose the first kilometre to a start button I forgot to press.
2. As a runner, I want no countdown before recording begins, so that I can start moving the instant I decide to.
3. As a runner, I want my warm-up walk recorded as a Walking Segment with its Route kept, so that the ground I covered is on the map but my Running time stays honest.
4. As a runner, I want the app to open a Running Segment on its own once I am actually running, so that I don't have to tell it.
5. As a runner, I want standing at a traffic light to become an Idle Segment after a short delay, so that waiting doesn't count as Running time.
6. As a runner, I want a Running Segment to resume automatically when I start moving again, so that auto-resume needs no attention from me.
7. As a runner, I want dropping from a run to a walk to become a Walking Segment, so that the two are separated without my input.
8. As a runner, I want brief wobbles at the boundary between running and walking not to shatter my Session into dozens of tiny Segments, so that the Segment list stays readable.
9. As a runner, I want to tap Force start when the app has me as walking but I am running, so that I can correct a misclassification immediately.
10. As a runner, I want to tap Force end when the app has me running but I have stopped, so that a hill I am walking up doesn't count as Running time.
11. As a runner, I want my Force start / Force end to hold for a short window before automatic classification can override it, so that my correction isn't undone a second later.
12. As a runner, I want recording to continue with the screen locked and the phone in my pocket, so that I can run normally.
13. As a runner, I want to wake the screen mid-run and see the map, distance, current pace, Running time, and Splits so far, so that I can check how I'm doing without stopping.
14. As a runner, I want to see at a glance whether the app currently has me as Running, Walking, or Idle, so that I can tell when it is wrong and correct it.
15. As a runner, I want to press stop when I've finished, so that the Session is written as one Workout Record.
16. As a runner, I want a confirmation before the Session is discarded or stopped by accident, so that a pocket tap can't end my run.

### Not having to remember to stop

17. As a runner, I want the Session to end itself once it exceeds a maximum distance, so that forgetting to stop can't produce an absurd Record.
18. As a runner, I want that distance limit to default to 42.195 km, so that no real run of mine is ever cut short by it.
19. As a runner, I want the Session to end itself once it exceeds a maximum duration, so that a Session left open overnight doesn't run until the battery dies.
20. As a runner, I want to set both limits in Settings, so that they match how far and how long I actually run.
21. As a runner, I want the Record produced by an automatic end to be saved normally, so that I keep the run rather than losing it to my own forgetfulness.

### GPS quality and gaps

22. As a runner, I want fixes with poor accuracy discarded rather than plotted, so that a bad fix doesn't add distance I never covered.
23. As a runner, I want a tunnel or an urban canyon to leave a gap in the Route rather than a straight line across the map, so that the map shows where I actually went.
24. As a runner, I want no distance accumulated across a GPS gap, so that a signal loss cannot inflate or deflate my total.
25. As a runner, I want recording to pick up from the gap when the signal returns, so that losing GPS costs me only the missing stretch.
26. As a runner, I want to be told when GPS has been unavailable for a while, so that I know the current numbers are incomplete.

### Reviewing history

27. As a runner, I want a list of every Workout Record with date, distance, Running time, and average pace, so that I can scan my history quickly.
28. As a runner, I want to open a Record and see its Route on a map, so that I can see where I ran.
29. As a runner, I want to see per-Split pace in the detail view, so that I can tell whether I faded.
30. As a runner, I want to see elevation gain for a Record, so that I can account for a hilly route when my pace looks slow.
31. As a runner, I want to see average cadence for a Record, so that I have a form metric to watch over time.
32. As a runner, I want to see how the Session broke down into Running, Walking, and Idle time, so that I can see how much of it was really running.
33. As a runner, I want to see the Segment sequence for a Session, so that I can check whether the classifier behaved sensibly.
34. As a runner, I want a helpful empty state before my first run, so that a fresh install doesn't look broken.
35. As a runner, I want to delete a Record that is wrong, so that bad data doesn't pollute my history.
36. As a runner, I want deletion to be confirmed, so that I don't lose a run to a mis-tap.

### Owning the data

37. As a runner, I want to export a Record as GPX, so that I can open my Route in any other tool that reads GPX.
38. As a runner, I want to export a Record as JSON containing everything the app stored, so that nothing is trapped in the app.
39. As a runner, I want exported coordinates to be raw WGS-84, so that the file is correct anywhere in the world and in any other tool.
40. As a runner, I want to share an export through the normal iOS share sheet, so that I can put it wherever I want.

### Surviving interruptions

41. As a runner, I want an in-progress Session to survive the app crashing or being killed, so that a long run isn't lost to a bug.
42. As a runner, I want to be offered the recovered Session when I reopen the app, so that I can continue it rather than starting fresh.
43. As a runner, I want everything to work with no network at all, so that trails and tunnels are not a problem.

### Permissions and settings

44. As a runner, I want to be told why the app needs always-on location before the system prompt appears, so that I understand what I am granting.
45. As a runner, I want to be told why the app needs motion access before that prompt appears, so that the request doesn't come out of nowhere.
46. As a runner, I want a clear explanation and a route to Settings if I have denied a permission the app needs, so that I can fix it without guessing.
47. As a runner, I want distances in kilometres and pace in min/km, so that the numbers match how I think about running.
48. As a runner, I want to adjust the classifier's thresholds in Settings, so that I can tune recognition against my own running while the logic is still being proven.

### Proving the classifier

49. As the developer, I want a debug mode that records raw location, motion, and cadence signals alongside the classifier's output during a real run, so that I have real-world material to test against.
50. As the developer, I want a recorded run to replay through the classifier offline and produce the same Segments every time, so that I can change the logic and see exactly what moved.
51. As the developer, I want my Force start / Force end taps captured in that recording as labels, so that the moments where the classifier was wrong are marked for later analysis.
52. As the developer, I want to export a recorded signal session, so that it can become a test fixture in the repo.

## Implementation Decisions

### `SessionProcessor` — the one seam

The heart of Phase 1 is a single pure module. It takes an ordered stream of timestamped signal events and returns one `WorkoutRecord`:

- **Input events**: location fix (latitude, longitude, altitude, horizontal accuracy, speed, timestamp); motion activity sample (`stationary` / `walking` / `running` / `cycling` / `automotive` plus confidence); cadence sample (steps per minute); force start; force end; stop.
- **Output**: a `WorkoutRecord` — its Segments, Splits, Route, and summary figures — plus the reason the Session ended (manual stop, distance limit, duration limit).
- **Configuration**: the classifier thresholds and the auto-end limits are passed in, not read from a global.

It has **no clock, no CoreLocation, no CoreMotion, no SwiftData, and no UI**. All time comes from event timestamps. Feeding it the same events twice produces the same Record, which is what makes record-replay testing possible.

Accuracy filtering, distance accumulation, GPS gap handling, the classification state machine, Split cutting, force-override precedence, and the auto-end limits all live behind this one boundary. None of them is a separately exposed interface.

### Layers outside the seam

- **Signal adapters**: wrap `CLLocationManager`, `CMMotionActivityManager`, `CMPedometer`, and `CMAltimeter` and emit the event stream. They contain no classification logic — they translate framework callbacks into events and nothing else.
- **Session store**: SwiftData. The local store is the single source of truth ([ADR-0005](adr/0005-local-first-storage.md)). Route points are a separate table written in batches, since a single Session can produce thousands.
- **Exporter**: the second, trivial seam — `WorkoutRecord` → GPX text, and `WorkoutRecord` → JSON text. Pure functions over the Record, with no I/O.
- **`MapProvider`**: the map sits behind a swappable interface ([ADR-0006](adr/0006-mapkit-and-wgs84-routes.md)). Phase 1 implements it with MapKit.
- **UI**: SwiftUI. Live Session screen (map, distance, current pace, Running time, Splits, current motion state, Force start / Force end, stop), history list, Record detail (map, Split pace chart, Segment breakdown), Settings.

### Classification

Three layers, per [ADR-0003](adr/0003-on-device-motion-classification.md):

1. **Recognition**: `CMMotionActivity` is the primary signal — Apple's on-device model, effectively free in battery terms.
2. **Fusion**: a state machine over Idle / Walking / Running combines that with GPS speed and cadence, applying hysteresis so boundary noise doesn't fragment the Session.
   - → Running: activity is `running` at medium-or-better confidence, or (activity uncertain) sustained speed above roughly 2.2 m/s with cadence above roughly 150 spm.
   - → Walking: activity is `walking`, or speed sustained in roughly 0.5–2.2 m/s.
   - → Idle: activity is `stationary` and speed below roughly 0.5 m/s, held for a dwell time of roughly 15–20 s.
   - Idle → previous state on detected movement.
   - Every threshold named here is configuration, not a constant, and is adjustable in Settings during Phase 1.
3. **Override**: Force start / Force end sets the state directly and locks it for a short window, during which layers 1 and 2 cannot change it.

Location fixes with horizontal accuracy worse than roughly 20–30 m are dropped before they reach the state machine or the distance total.

### Session lifecycle

A Session starts at app launch ([ADR-0001](adr/0001-open-to-record-manual-stop.md)) and ends on manual stop, or automatically when cumulative distance exceeds `maxDistanceKm` (default 42.195) or elapsed time exceeds `maxDuration` (default 6 hours). An automatically ended Session is saved as a normal Record, with the end reason retained.

In-progress state is checkpointed periodically so a crash or termination can be recovered and the Session offered back to the runner on relaunch.

### Data model

`WorkoutRecord`: id, startedAt, endedAt, timezone, totalDuration, runningDuration, walkingDuration, idleDuration, distance (metres), avgPace (s/km), elevationGain (m), avgCadence (spm), endReason, sourceDevice, appVersion.

- **Segments**: type (running / walking / idle), startedAt, endedAt, distance, avgPace.
- **Splits**: index, distance (1000 m except the last), duration, avgPace, elevationDelta.
- **RoutePoints**: timestamp, latitude, longitude, altitude, horizontalAccuracy, speed.

Route coordinates are stored and exported as WGS-84, always. Any datum conversion is a display concern only, and MapKit receives `CLLocation.coordinate` unconverted.

Segments tile the Session with no gaps. Splits are cut by cumulative distance and are orthogonal to Segments.

### Signal recording

A debug mode persists the raw event stream plus the classifier's output and any force overrides, and can export it. Exported recordings become replay fixtures. This is also the labelled corpus that a future Core ML classifier would be trained on, though training one is not part of this spec.

## Testing Decisions

**What a good test looks like here.** A test drives a module through its public boundary and asserts on what comes out. For `SessionProcessor` that means: feed an event stream, assert on the resulting Segments, Splits, Route, and totals. Tests must not reach for the state machine's internal state, the names of private helpers, or how many passes the implementation makes over the signals — all of that must stay free to change. If a behaviour cannot be observed in the returned Record, it is not a behaviour worth asserting.

**Modules under test.**

- **`SessionProcessor`** — the great majority of the test suite, since nearly all Phase 1 logic is behind it:
  - Segment classification, including hysteresis at boundaries, Idle dwell time, and auto-resume.
  - Force start / Force end precedence and the lock window.
  - Poor-accuracy fixes excluded from Route and distance.
  - GPS gaps: Route discontinuity preserved, no distance accrued across the gap, recording resumes after it.
  - Split cutting at kilometre boundaries, including the short final Split.
  - Summary arithmetic: Running time excludes Walking and Idle; durations sum to the Session span.
  - Auto-end on the distance limit and on the duration limit, with the correct end reason.
  - Degenerate inputs: no fixes at all, a Session that is entirely Idle, a stop arriving immediately after launch.
- **Exporter** — golden-file tests: a fixed Record renders to expected GPX and JSON, with coordinates verified as WGS-84 and unrounded.
- **Record-replay** — recordings captured from real runs replay to a stable, committed Record snapshot. These are the tests that prove the classifier against reality rather than against invented signals; a change in the snapshot is a change in classifier behaviour and must be reviewed as such.

**Prior art.** There is none — this is the first code in the repository. The first replay fixture and its snapshot therefore set the pattern for everything after, and should be built with that in mind: fixtures as plain committed data files, one Record snapshot each, no test-only hooks reaching inside the processor.

**Not unit-tested**: the signal adapters (thin translations of framework callbacks), SwiftData persistence, and SwiftUI views. Their correctness is a matter for real-device runs, not for the test suite.

## Out of Scope

Everything in Phase 2 ([ADR-0002](adr/0002-phased-delivery-free-cert-first.md)), which requires the paid Apple Developer membership:

- The Apple Watch app and the watch-anchored `HKWorkoutSession` ([ADR-0004](adr/0004-watch-anchored-workout-session.md)).
- Heart rate: live streaming, the detail-view curve, zone distribution, and calorie estimation.
- Writing `HKWorkout` to Apple Health and the Fitness rings.
- Lock-screen and Dynamic Island Live Activity.
- The Go backend and REST sync.
- Using the system's `motionPaused` / `motionResumed` events for auto-pause.

Also out of scope for this spec:

- Android or any non-Apple platform.
- Social features of any kind: feeds, friends, leaderboards, sharing to a community.
- Training plans, courses, coaching.
- Editing a Record after the fact, including trimming its start and end. Deleting the whole Record is the only correction offered.
- Sports other than running.
- Audio announcements during a run.
- Medals and achievements.
- Localisation. Chinese first; no second language.
- Health platforms other than Apple's.
- Imperial units. Metric is fixed for Phase 1.
- Training a custom Core ML classifier. Phase 1 only gathers the labelled corpus that would make it possible.

## Further Notes

**Deliberately accepted.** Recognition lags the true start of running by roughly 10–30 seconds. This is the known cost of having no start button, and a warm-up covers it in practice; the Route for those seconds is still recorded as Walking, so no ground is lost. Weekly re-signing under the free certificate is likewise accepted.

**Assumptions made in writing this spec, each worth a second look before implementation:**

- `maxDuration` defaults to 6 hours. The design draft left the number to be chosen from the runner's own pace and marathon cut-off times.
- An automatic end posts a local notification, so that the runner finds out the Session ended rather than discovering it later. This adds a notification permission prompt, which is why it is called out rather than assumed silently.
- The Split pace chart uses Swift Charts.
- Permission priming happens on first launch, before the system prompts, with copy explaining that always-on location is what allows recording to continue in a pocket. The exact copy and timing are still open.

**Vocabulary.** This spec uses the glossary in [`CONTEXT.md`](../CONTEXT.md). Notably, *pause* means only a deliberate user action and never the automatic transition into Idle; *Route* is the recorded GPS sequence while GPX is merely an export of it; and a *Session* is the live span whereas a *Workout Record* is what persists.

**Supersedes.** Sections 5–9 of [`docs/design-draft.md`](design-draft.md). Section 8 of that draft proposed three test seams (classifier, GPS/distance, Splits); this spec deliberately narrows that to one, since all three are observable at the `SessionProcessor` boundary.
