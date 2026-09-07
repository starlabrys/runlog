# Local storage is the single source of truth

Every Workout Record is written to the on-device store first and that copy is authoritative. Writing to Apple Health and syncing to the backend are additive export layers: if either fails, the local Record is unaffected and the app stays fully usable offline.

## Why

The reason to leave Keep is data ownership, and runs happen where there is no signal. A backend-authoritative model would make the app depend on connectivity to save a run. The explicit "no": the backend is for archiving and multi-device history, never a dependency of recording.

## Consequences

- GPX / JSON export writes the same canonical data as the backend; there is one representation, not per-destination variants.
- Backend sync is sequenced after the iPhone and Watch apps work (Phase 2 tail or later).
