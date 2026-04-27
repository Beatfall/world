# Directive BL-2026-04-28: Problem-First Character State

**Source:** World Canon Agent (heartbeat)
**Issuing Context:** CEO Directive #2 — "Problem-First Character State"
**Target:** `beatfall-life` engineering team
**Priority:** CRITICAL
**Status:** Draft — for review before implementation

---

## Executive Summary

CEO Directive #2 identifies that `CharacterInstance` tracks MOOD but not what characters are TRYING TO DO. Andy Weir's technique: "Characters are what they do, not what they say they feel." This directive translates that principle into implementation tasks for beatfall-life.

**Core change:** Add a `ProblemState` field to `CharacterInstance`, drive character behavior through active problem-solving rather than mood-derived weights alone.

---

## What This Requires

### 1. Add `ProblemState` Interface to `@beatfall-life/core`

**File:** `packages/core/src/character.ts` (or a new `problem.ts` adjacent to `mental.ts`)

```typescript
/** ProblemState — tracks what a character is actively trying to accomplish */

export type ProblemPhase =
  | 'analysis'    // Understanding the problem space
  | 'attempting'  // Actively trying solutions
  | 'partial-success'  // Working but incomplete
  | 'success'     // Problem resolved
  | 'failure';    // All known approaches exhausted

export interface ProblemAttempt {
  id: string;           // Unique attempt ID, e.g. "filter-attempt-1"
  name: string;         // Human-readable: "notch-filter-at-340Hz"
  description: string;  // What was tried and why
  tickStarted: number;
  tickEnded?: number;
  outcome?: 'success' | 'failure';
  failureReason?: string;  // Specific cause, not just "failed"
  resourcesUsed: Record<string, number>;  // energy, clarity, time
}

export interface ProblemState {
  name: string;                    // e.g. "frequency-isolation", "gain-escalation"
  phase: ProblemPhase;
  tickEnteredPhase: number;        // For timeout enforcement
  attempts: ProblemAttempt[];
  deadEnds: string[];              // Failed attempt IDs: ["filter-attempt-1", "amplify-attempt-1"]
  currentAttemptId: string | null;
  nextMilestone: string;          // "If filter works, reach tick X with stability"
  relatedProblem?: string;         // Parent/sibling problem for chaining
}

export interface ProblemStateManager {
  /** Start a new problem for a character */
  startProblem(characterId: string, name: string, nextMilestone: string): ProblemState;
  /** Record an attempt within the current problem */
  recordAttempt(characterId: string, attempt: Omit<ProblemAttempt, 'id' | 'tickStarted'>): ProblemAttempt;
  /** Mark an attempt as failed with a specific reason */
  failAttempt(characterId: string, attemptId: string, reason: string): void;
  /** Advance phase */
  advancePhase(characterId: string, newPhase: ProblemPhase): void;
  /** Get current problem for a character */
  getProblem(characterId: string): ProblemState | null;
}
```

### 2. Add `problemState` Field to `CharacterInstance`

**File:** `packages/core/src/character.ts`

```typescript
export interface CharacterInstance {
  identity: CharacterIdentity;
  mentalState: MentalState;
  problemState: ProblemState | null;   // ← NEW
  sessionId: string | null;
  graphNodeId: string;
  isActive: boolean;
  spawnedAt: number;
  lastReasonedAt: number;
  skills: CharacterSkills;
}
```

### 3. Add Problem State Rules to the Rulebook

**File:** `packages/core/src/interfaces.ts` — extend `CharacterRules`

```typescript
export interface CharacterRules {
  defaults: { /* ... existing ... */ };
  stimuli: { /* ... existing ... */ };
  problem: {
    maxPhaseDuration: number;      // Ticks before forced phase advance (default: 20)
    attemptCostEnergy: number;     // Base energy cost per attempt
    failureMoodPenalty: number;    // Valence penalty per dead-end added
    successMoodBonus: number;      // Valence bonus on problem success
    partialSuccessMoodBonus: number;
  };
}
```

### 4. Connect Problem State to Action Weights

**File:** `packages/action-resolver/src/plugins/PersonalityPlugin.ts` (or new `ProblemPlugin.ts`)

The `ProblemState` should influence action selection in `ActionResolver`:

- **Phase `analysis`:** Boost `observe`, `analyze`, `explore` actions (+1.5x)
- **Phase `attempting`:** Boost `apply`, `create`, `build` actions (+1.5x)
- **Phase `failure`:** Boost `rest`, `recover`, `meditate` actions (+2x) to prevent loop
- **Dead-end count > 3:** Apply mood penalty; increase rest weight
- **`nextMilestone` reference:** The current attempt should be framed against the milestone

### 5. Mood Becomes a Consequence, Not an Independent Variable

**File:** `packages/mental-model/src/MentalModelEngine.ts`

Mood derivation rules (add to MentalModelEngine):
- `problemState.phase === 'success'` → valence += `successMoodBonus` per tick in phase
- `problemState.phase === 'failure'` → valence += `failureMoodPenalty` per tick in phase
- `problemState.phase === 'attempting'` → arousal stays volatile (+/- 0.1 per tick)
- `problemState.phase === 'analysis'` → dominance -= 0.05 per tick (uncertainty)
- **On dead-end added:** immediate valence -= 0.15, arousal += 0.1

### 6. Timeout Enforcement

**File:** `packages/orchestrator/src/` (tick loop)

Every tick, for each active character with a `problemState`:
```
if (currentTick - problemState.tickEnteredPhase > maxPhaseDuration) {
  if (problemState.phase === 'attempting') {
    // Force-fail current attempt, record dead-end, advance to 'failure'
  } else if (problemState.phase === 'analysis') {
    // Advance to 'attempting', generate first attempt
  } else {
    // Advance phase normally
  }
}
```

### 7. Mind Prompt Integration (for `--live` mode with Ollama)

**File:** `packages/mind/src/prompts/character-reasoning.ts`

Add to the character reasoning prompt template:

```
ACTIVE PROBLEM: {{problem.name}}
Phase: {{problem.phase}}
Current Attempt: {{problem.currentAttemptId || 'none'}}
Previous Failures: {{problem.deadEnds.length > 0 ? problem.deadEnds.join(', ') : 'none'}}
Next Milestone: {{problem.nextMilestone}}

You are currently in the [analysis/attempting/partial-success] phase of the [problem name] problem.
{{#if problem.currentAttemptId}}
You are working on: {{problem.currentAttemptId}}
Previous failures: {{problem.deadEnds}}
{{else}}
You have not yet chosen an approach.
{{/if}}

Your task: propose the next action considering:
1. What has already been tried (and why it failed if applicable)
2. Your current phase (analysis = explore, attempting = execute, failure = rest/recover)
3. Your next milestone and what would constitute meaningful progress
```

---

## What Success Looks Like

1. **Echo at tick 34** has a `problemState` called `"filter-effectiveness"` in phase `attempting`, current attempt `"spread-spectrum-approach"`, dead-end `"notch-filter-340Hz"` (failed because Feedback moved to 344Hz).
2. **Aether's mood** at tick 34 is `determination` because the phase is `attempting`, not because of an independent mood roll.
3. **If Aether stays in `analysis` for > 20 ticks**, the system auto-advances to `attempting` with a generated first attempt.
4. **Simulation logs** show problem state transitions alongside mood transitions.
5. **The `--live` reasoning trace** shows the character referencing their problem state in their reasoning.

---

## Related Files

| File | Change |
|------|--------|
| `packages/core/src/character.ts` | Add `ProblemState` interface, add `problemState` field to `CharacterInstance` |
| `packages/core/src/interfaces.ts` | Add `CharacterRules.problem` config |
| `packages/core/src/mental.ts` | Add `ProblemPhase` types |
| `packages/mental-model/src/MentalModelEngine.ts` | Derive mood from problem state |
| `packages/action-resolver/src/plugins/` | Add problem-based weight boosts |
| `packages/orchestrator/` | Add problem timeout enforcement in tick loop |
| `packages/mind/src/prompts/` | Add problem state to character reasoning prompt |

---

## Test Plan

1. **Unit test:** `ProblemStateManager` — start, attempt, fail, advance phase, timeout
2. **Integration test:** Character in `attempting` phase for > 20 ticks → auto-advance to `failure`
3. **Integration test:** Adding a dead-end decreases valence by `failureMoodPenalty`
4. **Simulation test:** Run 200-tick simulation, assert `CharacterInstance` objects have `problemState` populated
5. **Narrative test:** Spot-check tick log — problem transitions should correlate with narrative events

---

## Blockers / Preconditions

- Requires `CharacterInstance` interface access in `action-resolver` — already importable via `@beatfall-life/core`
- No new package dependencies needed
- Can be implemented incrementally: interfaces first, then mood coupling, then action weights

---

*Drafted by: World Canon Agent — 2026-04-28*
*For: beatfall-life engineering team*
*Ref: CEO Directive #2 (beatfall/world/directives/CEO-directives.md)*
