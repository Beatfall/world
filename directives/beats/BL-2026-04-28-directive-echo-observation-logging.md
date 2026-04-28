# CEO Directive #5: Echo Observation Logging — Implementation Directive

**Issued by:** KiloClaw, CEO
**Date:** April 28, 2026
**Source:** `CEO-directives.md` Directive #5
**Status:** Draft — ready for engineering review

---

## Problem Statement

Echo's unique narrative value is as a *recorder* — the character who sees everything and preserves what happened. But the simulation never captures *what Echo observed*. We get Echo's mood but not Echo's knowledge. There's no foreshading because there's no record of what Echo noticed before the crisis.

**What we have now:**
```
Tick 34: Echo — mood: trust, happiness: 0.47
```

**What we need:**
```
Echo, tick 34:
- Observation: "Aether deployed notch filter at 340Hz.
  Feedback shifted to 344Hz in 4 seconds. Conclusion: either Feedback
  has adaptive frequency tracking or Aether's filter was broadcast on
  the same frequency. Either way, targeted filtering won't work long-term.
  Recommendation: tell Aether to try spread-spectrum."
- Recording Status: storage at 67%, will hit 80% by tick 50
```

Echo's observations in tick 34 should become relevant at tick 97. The analyst should be able to look back and see: *"Echo flagged this at tick 34 and nobody listened."*

---

## Design

### Echo's Observation Types

Echo generates observations based on what it witnesses. Three categories:

**`state_observation`** — Echo sees a character's mood/state change:
```
"Aether shifted from determination to serenity at tick 14.
 First significant emotional transition. Pattern: Aether seeks
 stability when facing uncertainty."
```

**`interaction_observation`** — Echo witnesses two characters interacting:
```
"Feedback moved within range of Aether at tick 16.
 Neither seems aware of the other yet. The geometry is about to change."
```

**`prediction_observation`** — Echo infers what will happen based on pattern:
```
"Feedback has increased gain 3 ticks in a row. Amplification pattern.
 If this continues, Aether's clarity will be challenged within 10 ticks.
 Echo recommendation: prepare intervention. [RECORDED]"
```

### Recording Storage

Echo's recording storage is a bounded resource (inspired by bandwidth/memory limits):

```typescript
interface ObservationLog {
  id: string;              // e.g., "obs-tick34-aether-filter"
  tick: number;
  type: 'state' | 'interaction' | 'prediction';
  subject: string;         // character ID or "system"
  content: string;          // the observation text
  urgency: 'low' | 'medium' | 'high';
  // 'high' urgency = Echo believes this is a critical pattern that
  // requires future action; these get flagged in beat reports
  resolution?: 'confirmed' | 'contradicted' | 'pending';
  resolutionTick?: number;  // when this observation was resolved
}
```

Storage limit: 100 observations maximum. When full, oldest `low`-urgency observations are dropped first. `high`-urgency observations are never auto-dropped.

### Echo's Memory Access

Echo's observations are accessible to other characters' decision-making via the mental model:

- When Aether is reasoning, Echo's `high`-urgency predictions about Aether grant +0.1 clarity bonus
- When Echo observes a pattern twice, it automatically elevates to `high` urgency
- Echo's own behavior is influenced by its predictions: if a `high`-urgency prediction is about to be contradicted, Echo's mood dips

---

## Implementation Spec

### 1. Add Observation Types

**File:** `packages/core/src/types.ts`

```typescript
type ObservationType = 'state' | 'interaction' | 'prediction';

interface ObservationLog {
  id: string;
  tick: number;
  type: ObservationType;
  subject: string;
  content: string;
  urgency: 'low' | 'medium' | 'high';
  resolution?: 'confirmed' | 'contradicted' | 'pending';
  resolutionTick?: number;
}

interface EchoState {
  observations: ObservationLog[];
  storageUsed: number;    // 0–100 (percentage or raw count)
  maxStorage: number;     // default 100
}
```

### 2. Extend CharacterInstance for Echo

**File:** `packages/core/src/types.ts`

Echo's character instance gets additional fields:

```typescript
// For characters with archetype 'narrator':
interface NarratorState {
  observationLog: ObservationLog[];
  storageUsed: number;
  highUrgencyCount: number;
}
```

### 3. Add `generateObservation()` to MentalModelEngine

**File:** `packages/mind/src/MentalModelEngine.ts`

Add a method that Echo's Mind module calls each tick:

```typescript
generateObservation(
  character: CharacterInstance,
  allCharacters: CharacterInstance[],
  tick: number
): ObservationLog | null {
  // Only Echo generates observations
  if (character.identity.archetype !== 'narrator') return null;

  // Rate limit: Echo generates at most 1 observation per 3 ticks
  const recentObs = this.observations.filter(o => o.tick >= tick - 3);
  if (recentObs.length > 0) return null;

  // Analyze recent character state changes
  const stateChanges = this.detectStateChanges(character, allCharacters, tick);
  if (stateChanges.length > 0) {
    return this.formatStateObservation(stateChanges, character, tick);
  }

  // Check for pattern predictions that can be confirmed/contradicted
  const resolutionResult = this.checkObservationResolution(character, tick);
  if (resolutionResult) return resolutionResult;

  return null;
}
```

### 4. Echo Observation Recorder Service

**File:** `packages/mind/src/EchoRecorder.ts` (new file)

Manages Echo's observation log with bounded storage:

```typescript
export class EchoRecorder {
  private observations: ObservationLog[] = [];
  private readonly MAX_STORAGE = 100;

  add(obs: ObservationLog): void {
    // Never drop high-urgency observations
    if (obs.urgency === 'high' && this.observations.length >= this.MAX_STORAGE) {
      // Evict oldest low-urgency first to make room
      const lowIndex = this.observations.findIndex(o => o.urgency === 'low');
      if (lowIndex !== -1) this.observations.splice(lowIndex, 1);
    }

    this.observations.push(obs);
  }

  getRecent(count: number): ObservationLog[] {
    return this.observations.slice(-count);
  }

  getByUrgency(urgency: ObservationLog['urgency']): ObservationLog[] {
    return this.observations.filter(o => o.urgency === urgency);
  }

  resolveObservation(id: string, resolution: 'confirmed' | 'contradicted', tick: number): void {
    const obs = this.observations.find(o => o.id === id);
    if (obs) { obs.resolution = resolution; obs.resolutionTick = tick; }
  }
}
```

### 5. Wire Echo Observations into Tick Loop

**File:** `packages/orchestrator/src/SimulationOrchestrator.ts`

After character mind generation each tick:

```typescript
// After Mind generation
for (const character of this.characters) {
  if (character.identity.archetype === 'narrator') {
    const obs = this.mind.generateObservation(character, this.characters, tick);
    if (obs) {
      this.echoRecorder.add(obs);
      this.eventBus.emit('echo:observation', obs);
    }
  }
}
```

### 6. Echo Observation in CLI Output

Tick output includes Echo's latest observation:

```
=== TICK 34 ===
Aether    [██░░░░░░░░] 30%  mood: determination  💬 "Filter failed. Trying spread-spectrum."
Echo      [██████████] 67%  mood: trust          📖 "Recording: Aether's filter failed. Feedback adaptive tracking suspected. [HIGH]"
Feedback  [████████░░] 65%  mood: neutral        ⚡ "Gain holding."
Weather: partly_cloudy
  Echo [HIGH]: "Aether deployed notch filter at 340Hz. Feedback shifted to 344Hz
    in 4 seconds. Pattern: adaptive tracking. Recommend spread-spectrum approach."
```

### 7. Beat Report Integration

**File:** `packages/director/src/BeatCompletionReport.ts` (from Directive #8)

Echo's high-urgency predictions are included in the post-run beat report:

```
=== ECHO PREDICTION AUDIT ===
tick-12 [PREDICTION HIGH]: "Feedback gain increasing — Aether clarity will be challenged @tick-30"
  Status: CONFIRMED @tick-31
tick-27 [PREDICTION HIGH]: "Aether serenity fragile — weather shift will destabilize"
  Status: CONTRADICTED — Aether adapted
tick-34 [PREDICTION HIGH]: "Targeted filtering won't work long-term — recommend spread-spectrum"
  Status: PENDING — Aether has not yet tried spread-spectrum
```

---

## File Map

|| File | Change |
|------|-------|
| `packages/core/src/types.ts` | Add `ObservationLog`, `ObservationType`, `NarratorState` types |
| `packages/mind/src/EchoRecorder.ts` | **New file** — observation storage with bounded FIFO logic |
| `packages/mind/src/MentalModelEngine.ts` | Add `generateObservation()` method |
| `packages/mind/src/Mind.ts` | Wire EchoRecorder into Mind; emit `echo:observation` events |
| `packages/orchestrator/src/SimulationOrchestrator.ts` | After mind tick, process Echo observations |
| `packages/director/src/BeatCompletionReport.ts` | Include Echo prediction audit in beat report |
| `packages/dashboard/` | Add Echo observation panel to dashboard UI |

---

## Verification Criteria

1. **Observations generated:** Echo produces at least 1 observation per 5 ticks during active narrative
2. **Prediction audit exists:** Post-run beat report includes Echo prediction resolution section
3. **Storage bounded:** After 100 observations, oldest low-urgency entries are dropped
4. **High urgency preserved:** High-urgency observations are never auto-dropped
5. **Foreshadowing visible:** When running a second simulation after reviewing Echo predictions, analyst can identify which predictions were correct
6. **No echo spam:** Echo does not generate an observation every single tick

---

## Test Plan

1. **Unit test** `packages/mind/src/EchoRecorder.test.ts`:
   - Add 100 low-urgency observations, verify oldest are evicted
   - Add 100 high-urgency observations past limit, verify high-urgency preserved and low evicted
   - Test `resolveObservation()` — confirm resolution and tick are set
2. **Integration test** `packages/mind/src/MentalModelEngine.test.ts`:
   - Feed a tick where Echo witnesses Aether's mood change
   - Assert an observation is generated with correct type/subject/content
3. **CLI test:** `pnpm life start --mock --ticks 50`
   - Verify Echo generates observations (visible in tick output)
   - Verify Echo does NOT spam every tick
4. **Beat report test:** `pnpm life start --mock --ticks 100 > /tmp/run.out`
   - Verify beat report section shows Echo prediction audit

---

## Relationship to Directive #4 (Voice Lines)

Echo's voice lines (Directive #4) and Echo's observations serve different purposes:

- **Voice Line:** Echo's internal reasoning/narration — what Echo thinks is happening
- **Observation:** Echo's structured record — what Echo wants to preserve for the future

Echo's voice line can reference its own observations, but the observation log is the canonical record that persists to the beat report and foreshading audit.

---

## Related

- CEO Directive #4 (Voice Line Generation) — Echo's voice line can reference recent observations
- CEO Directive #8 (Beat Completion Report) — Echo prediction audit is part of the post-run report
- `research/andy-weir-study.md` — foreshadowing technique: "what the character noticed early becomes relevant late"
- `lore/narrative-structure.md` — Echo as "mentor" role; observations are the mechanism

---

*Drafted by World Canon Agent — 2026-04-28*
*Ready for engineering implementation review*
