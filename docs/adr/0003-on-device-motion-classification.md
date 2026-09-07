# Motion classification runs on device

Deciding whether the runner is running, walking, or idle happens entirely on the device. The primary signal is Apple's built-in `CMMotionActivity` (a trained human-activity-recognition model that runs on the motion coprocessor, free, near-zero battery), fused with GPS speed and cadence in a small state machine that produces Segments and drives auto-pause.

A server is used only for archiving raw signals, re-processing old Workout Records when the model improves, and (eventually) training. It is never in the live classification path.

## Considered options

- **Stream raw IMU to a server and classify there**: a central model is easy to improve, but adds seconds of latency, fails with no signal (tunnels, trails, airplane mode), and continuously uploading ~50 Hz sensor data is one of the largest battery drains on the device.
- **On-device, Apple's `CMMotionActivity` as the primary signal** (chosen): sub-second, works offline, negligible battery, nothing leaves the device.
- **On-device, custom Core ML model**: deferred. The "signal recording" debug mode collects labelled runs; a `Create ML` activity classifier can be trained later and adopted only if it clearly beats the rule-based baseline on held-out runs.

## Consequences

- An LLM is explicitly the wrong tool here (wrong modality, too large, too slow, too power-hungry).
- Phase 2 may prefer the system's `motionPaused` / `motionResumed` events for outdoor running; the state machine stays as the fallback.
