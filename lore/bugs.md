# Beatfall Life — Canonical Bug Registry

> **Purpose:** Document known simulation bugs that affect narrative outcomes, with symptoms, cause analysis, and verification criteria. Referenced by timeline.md Section VIII and the Beat Completion Assessment.

---

## Bug #120 — Premature Shadow Elimination (CRITICAL)

**First Observed:** v1 (tick 35), reproduced in v3 (tick 97), v5d/v5e (tick 35)
**Status:** 🐛 Open — no simulation run has completed a full Shadow arc

### Symptom

Feedback (the Shadow antagonist) is eliminated before it can be defeated in a proper climax. In all v0.1.0-family runs, Feedback reaches peak emotional state (ecstasy) and is then immediately eliminated — the Shadow is destroyed at the moment of triumph, not after a descent.

### Root Cause

The elimination gate in `CharacterManager` fires when `safety` drops below a threshold, without checking whether the Shadow has completed its arc. The `StoryBeatDirector` has no veto over eliminations that occur before a `shadow-peak` beat has fired and the Shadow has begun its road-back.

### Affected Runs

| Run | Feedback Eliminated | Shadow-Peak Beat Fired? |
|-----|---------------------|------------------------|
| v1-basic | Tick 35 | ❌ No — eliminated at peak |
| v2-emotional-intelligence | Tick 176 | ✅ Yes — at tick ~100 (200-tick run) |
| v3-autonomous-npcs | Tick 97 | ❌ No |
| v5d-latest | Tick 35 | ❌ No |
| v5e-pr149-plugin-architecture | Tick 35 | ❌ No |
| v5f-6pr-merged | Tick ~35 (inferred) | ❌ No |
| v5h-5new-commits | Tick ~35 (inferred) | ❌ No |

**Note:** v2 is the only run where the Shadow was present at its own climax (tick ~100) before being defeated. This is the correct narrative order.

### Narrative Impact

- Hero's Journey Stage 8 (The Ordeal) is hollow — the Shadow is cut off at its apex
- The `all-is-lost` beat cannot fire because the Shadow is gone before the crisis moment
- The `resolution` beat never fires because the story cannot complete its arc
- Aether's joy at tick 100 is unearned — there was no true confrontation

### CEO Directive Coverage

- **Directive #1 (Shadow Anti-Suppression)** — CRITICAL — draft complete. Implements `shadowPeakFired` flag and `canEliminateCharacter()` gate.
- **Directive #3 (Director Beat Injection)** — HIGH — draft complete. Director beat events must fire before eliminations are permitted.

### Verification Criterion (v0.2.0)

A simulation run is considered to have fixed Bug #120 when:
1. ✅ Feedback is present at tick 100 (storm climax)
2. ✅ `shadow-peak` beat fires before any elimination of Feedback
3. ✅ `all-is-lost` beat fires (requires Shadow at peak, Hero at minimum)
4. ✅ Feedback is eliminated AFTER `shadow-peak` and AFTER `all-is-lost` have both fired
5. ✅ `resolution` beat fires in a run that reaches tick 200+

---

## Bug #119 — Mood-Driven Weather (OPEN)

**First Observed:** v5h-5new-commits
**Status:** 🐛 Open

### Symptom

Weather cycles every ~20 ticks regardless of narrative progression. The `storm` at tick 100 arrives on a fixed timer, not as a consequence of narrative climax. Weather appears to be driven by aggregate mood state (time-driven mood cycles) rather than by story events.

### Cause Analysis

Weather transitions are triggered by `MoodGradient` mood-driven cycles, not by narrative beat progression. When `averageMood > threshold`, weather advances. This means weather can resolve to `storm` before the narrative climax has occurred.

### Evidence

v5h report: "Weather still cycles every ~20 ticks; character presence restored to 100%." The weather arc completes regardless of whether the narrative arc is complete.

### Relationship to Bug #120

Bug #120 and Bug #119 are independent but both contribute to hollow climaxes. Even if Bug #120 were fixed (Shadow present at climax), weather would still arrive on a fixed timer rather than being driven by the confrontation itself.

### CEO Directive Coverage

- **Directive #2 (Problem-First State)** — CRITICAL — draft complete. Mood-as-consequence model would tie mood transitions to problem-solving outcomes rather than time.

---

## Bug #110 — Character Location Absent (OPEN)

**First Observed:** v0.1.0 (and all prior runs)
**Status:** 🐛 Open

### Symptom

90%+ of tick snapshots show character location as "unknown". Characters appear to lack spatial positioning in the simulation world despite 5 canonical locations being defined (loc_0 through loc_4).

### Evidence

v0.1.0 report: "All characters lack location data — 'unknown' for 90%+ of ticks."

### Canonical Locations (Reference)

| ID | Name | Frequency Band |
|----|------|---------------|
| loc_0 | The Source | full spectrum |
| loc_1 | The Midfield | vocal range |
| loc_2 | The Low Shelf | bass range |
| loc_3 | The High Pass | high frequency |
| loc_4 | The Notch | presence range |

---

*Maintained by: World Canon Agent*
*Source: Simulation run reports, timeline.md Section VIII*
*Last updated: 2026-04-29T01:15:00Z*
