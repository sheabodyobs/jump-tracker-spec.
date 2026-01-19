# jump-tracker.spec — v0.1.0 (draft)

## 0. Summary
Goal: detect takeoff/landing events and compute jump metrics (especially GCT) from single-camera video.

Primary outputs:
- takeoff_times[]
- landing_times[]
- gct_seconds[]
- confidence + notes

---

## 1. Definitions

Time conventions:
- All timestamps `t` are seconds from video start (t=0.0).
- Frame indices are 0-based.
- `t ≈ frame / fps` (nearest-frame mapping).

Phases:
- **Contact**: at least one foot is contacting the ground.
- **Flight**: both feet are off the ground.
- **Takeoff**: transition Contact → Flight.
- **Landing**: transition Flight → Contact.

Metrics:
- **GCT (ground contact time)**: duration of the Contact phase immediately preceding a Takeoff for a detected jump.

---

## 2. Inputs

### 2.1 Input object (logical)
Required:
- `video_uri` (string): local path or remote URL
- `fps` (number > 0)
- `width` (int > 0)
- `height` (int > 0)

Optional:
- `analysis_id` (string): caller-provided id
- `athlete_id` (string)
- `session_id` (string)
- `camera_view` (enum): `side` | `front` | `rear` | `unknown`
- `notes` (string)

### 2.2 Preconditions
- Athlete is visible for analyzed segments.
- Video quality is sufficient to estimate foot contact most of the time.
- If preconditions fail, implementation MUST still return an output with low confidence and notes (not crash).

---

## 3. Outputs

### 3.1 Output object (logical)
Required:
- `spec` (string): `jump-tracker/0.1.0`
- `video_uri` (string): echo input
- `analysis_id` (string): generated if not provided
- `events` (array<Event>)
- `metrics` (object)
- `confidence` (object)

Optional:
- `artifacts` (array<object>): references to overlays or debug outputs

### 3.2 Event schema
Each event MUST include:
- `type` (enum): `contact_start` | `contact_end` | `takeoff` | `landing`
- `t` (number)
- `frame` (int)
- `side` (enum): `left` | `right` | `both` | `unknown`
- `confidence` (number in 0..1)

Ordering rules:
- `contact_start` MUST occur before its matching `contact_end`.
- `takeoff` MUST align with `contact_end` of a contact phase that transitions into flight.
- `landing` MUST align with `contact_start` of a contact phase that follows flight.

Completeness rules:
- If the system detects a flight segment, it SHOULD emit a `takeoff` and `landing` (even if low confidence).
- If no reasonable estimate exists, omit the event but add a `confidence.notes` entry explaining why.

### 3.3 Metrics schema
Required:
- `jumps_detected` (int >= 0)
- `takeoff_times` (array<number>)
- `landing_times` (array<number>)
- `gct_seconds` (array<number|null>)
- `flight_seconds` (array<number|null>)

Constraints:
- Arrays MUST be same length (`jumps_detected`) and ordered chronologically.
- `gct_seconds[i]` corresponds to `takeoff_times[i]`.
- `flight_seconds[i] = landing_times[i] - takeoff_times[i]` when both exist.

Optional:
- `foot_contact_pattern` (array<enum>): `forefoot` | `midfoot` | `rearfoot` | `unknown`
- `view` (enum): echo camera view if estimated
- `notes` (array<string>): non-confidence notes (e.g., filtering decisions)

---

## 4. Jump detection rules (semantic)

### 4.1 What counts as a jump
A “jump” MUST be defined as:
- one `takeoff` followed by one `landing`
- with `flight_seconds > 0.05` (reject noise)
- within a continuous visible segment

If multiple hops occur rapidly, each takeoff/landing pair is a separate jump.

### 4.2 GCT computation
For each jump i:
- Let `contact_start_i` be the start time of the Contact phase immediately preceding `takeoff_i`.
- `gct_i = takeoff_i - contact_start_i`

If `contact_start_i` is missing or ambiguous:
- set `gct_i = null`
- add a note in `confidence.notes`

### 4.3 Ambiguity handling
If takeoff frame is ambiguous by ±k frames:
- emit `takeoff` with `confidence < 0.5`
- include a note: `"ambiguous_takeoff_window: ±k frames"`

---

## 5. Confidence reporting

### 5.1 Required fields
`confidence` MUST include:
- `global` (0..1)
- `by_metric` (object):
  - `takeoff`
  - `landing`
  - `gct`
  - `flight`
  - `foot_contact_pattern` (if provided)
- `notes` (array<string>)

### 5.2 Example notes
- `occlusion: feet not visible around landing`
- `motion_blur: high blur during contact boundary`
- `camera_view_unknown: heuristics may be degraded`

---

## 6. Acceptance tests (spec-level)

A conforming implementation MUST be able to pass the following tests:

1. **Ordering**
   - Events are in non-decreasing time order.
   - For any jump, takeoff precedes landing.

2. **GCT correctness**
   - For each i where `gct_seconds[i] != null`:
     `abs(gct_seconds[i] - (takeoff_times[i] - contact_start_i)) <= 1/fps`

3. **Flight correctness**
   - For each i where `flight_seconds[i] != null`:
     `abs(flight_seconds[i] - (landing_times[i] - takeoff_times[i])) <= 1/fps`

4. **Schema declaration**
   - Output includes `spec == "jump-tracker/0.1.0"`.

5. **Ambiguity**
   - When takeoff is ambiguous, `confidence.by_metric.takeoff < 0.5` and at least one `confidence.notes` entry exists describing the ambiguity.

6. **No-crash contract**
   - If preconditions fail, output still returns a well-formed object with `confidence.global < 0.5` and at least one note.
