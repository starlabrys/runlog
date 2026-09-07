# Open the app to record; stopping is the only manual step

Opening runlog *is* the intent to exercise. There is no "start" button and no 3-2-1 countdown: a Session begins the moment the app launches, GPS and motion monitoring run from then on, and the app classifies the runner into Running / Walking / Idle Segments by itself. The runner only ever presses "stop".

This is the product's whole reason to exist, so it is hard to reverse: the data model, the classifier, and every screen assume it.

## Considered options

- **Conventional explicit start + countdown** (Keep, Nike Run Club, Strava): precise start instant, no misclassification at the boundary, but the runner can forget to start and loses the first kilometre.
- **Open to record, classify after the fact** (chosen): never lose the start; the cost is a roughly 10-30 second lag before "running" is recognised and the occasional misclassified boundary.

## Consequences

- A warm-up jog covers the recognition lag in practice, so the lag is acceptable.
- A manual **Force start / Force end** override is required as a safety valve for when the classifier is wrong (e.g. a slow jog read as walking).
- "When did the Session start" is unambiguous (app launch); "when did *running* start" is a classifier output that can be corrected, never a user input.
