# runlog

runlog records running workouts on iPhone (and, later, Apple Watch). It decides on its own when the runner is running, walking, or standing still, so the language below draws hard lines between those three states, and between the *live span* of activity and the *saved result*.

## Language

### A session and its parts

**Session**:
The continuous span from opening the app to the runner manually stopping. Produces exactly one Workout Record.
_Avoid_: workout, activity, tracking

**Segment**:
A contiguous stretch within a Session classified as a single motion state: Running, Walking, or Idle. Segments cover the whole Session with no gaps.
_Avoid_: interval, phase, lap

**Idle**:
A Segment where the runner is judged stationary: a traffic light, a tied shoelace, a rest. Idle time counts toward neither Running time nor Walking time.
_Avoid_: pause (reserve that for a deliberate user action), stopped, break

**Warm-up**:
The slow walking or jogging before real running starts. Recorded as ordinary Walking Segment(s): its Route is kept, but it never becomes Running time.
_Avoid_: pre-run, prep

**Running time**:
The sum of every Running Segment's duration in a Session. This is the headline duration shown to the runner.
_Avoid_: elapsed time, total time (those also count Walking and Idle)

### Automatic vs. manual control

**Auto-pause / auto-resume**:
The Session moving itself into an Idle Segment when motion stops, and back out when it resumes, with no user action.
_Avoid_: pause (a deliberate user action)

**Auto-end**:
The Session ending on its own, with no stop press, after a set number of continuous minutes with no Running Segment. Distinct from auto-pause, which only moves the Session into Idle and lets it keep going.
_Avoid_: timeout, auto-stop

**Force start / Force end**:
A control the runner taps to override the classification and begin or end a Running Segment immediately, whatever the motion signals say.
_Avoid_: manual lap, manual pause

### The recorded output

**Route**:
The ordered sequence of GPS fixes recorded across a Session (position, elevation, time). Where GPS was lost, the Route has a gap, not a straight line.
_Avoid_: track, path, trace, GPX (GPX is an export format, not the Route)

**Split**:
A one-kilometre slice of the Session by cumulative distance, used for pace analysis. Orthogonal to a Segment: a Split is cut by distance, a Segment by motion state.
_Avoid_: lap, kilometre marker, mile

**Workout Record**:
The single entry saved when a Session ends: its Segments, Splits, Route, and summary figures.
_Avoid_: activity, entry, log, session (the Session is the live span; the Record is what persists)
