# Directive BL-2026-04-28: StorylineDirector Beat Injection

**Source:** World Canon Agent (heartbeat)
**Issuing Context:** CEO Directive #3 — "StorylineDirector Beat Injection"
**Target:** `beatfall-life` engineering team
**Priority:** HIGH
**Status:** Draft — for review before implementation

---

## Executive Summary

CEO Directive #3 requires that `StorylineDirector` and `StoryBeatDirector` — already implemented in `packages/director/` — actually fire narrative beat events into the simulation tick loop. Currently, the Director generates beat plans but the beat events never modify character behavior or simulation state. The CLI runs in mock mode without any director influence.

**Core deliverable:** Director beats fire on the orchestrator event bus, modify character mental models, and are visible in CLI output. `--director` flag enables beat injection.

---

## What This Requires

### 1. Understand the Existing Architecture

**`packages/director/src/StorylineDirector.ts`** — Macro-level director using a smarter model to plan story phases and inject events. Already has:
- `DirectorPhase` enum: `'setup' | 'conflict' | 'climax' | 'resolution'`
- `planStoryline()` — generates a phase plan
- `shouldInjectEvent()` — per-tick probability decision

**`packages/director/src/StoryBeatDirector.ts`** — Beat-level director tracking which narrative beats have fired. Already has:
- `BeatPhase` enum: `'ordinary-world' | 'call' | 'tests' | 'ordeal' | 'reward' | 'road-back' | 'resurrection'`
- `tick()` — advances beat state machine
- `getPhase()` — returns current phase

**`packages/orchestrator/src/index.ts`** — Orchestrator tick loop. Already has an event bus (`EventBus`). Currently does NOT subscribe to director beat events.

**The gap:** Beat events exist in the director classes but are never subscribed to by the orchestrator, and their effects are never applied to character mental models.

---

### 2. Define Beat Event Types

**File:** `packages/core/src/director-events.ts` (new)

```typescript
export type DirectorBeatType =
  | 'inciting-incident'
  | 'shadow-rise'
  | 'all-is-lost'
  | 'shadow-peak'
  | 'beatfall'
  | 'resolution';

export interface DirectorBeatEvent {
  type: 'director:beat';
  beat: DirectorBeatType;
  tick: number;
  data?: {
    phase?: string;
    [key: string]: unknown;
  };
}

export interface DirectorDirectiveEvent {
  type: 'director:directive';
  directive: string;       // e.g. 'feedback:gain-floor+0.2'
  targetId?: string;       // character ID, or undefined for global
  duration?: number;       // ticks
  tick: number;
}

export type DirectorEvent = DirectorBeatEvent | DirectorDirectiveEvent;
```

---

### 3. Emit Beat Events from StoryBeatDirector

**File:** `packages/director/src/StoryBeatDirector.ts`

The director already has `BeatPhase` state machine. Add beat-firing:

```typescript
import type { DirectorBeatEvent } from '@beatfall-life/core';

// Inside StoryBeatDirector
private firedBeats: Set<string> = new Set();

public tick(currentTick: number, totalTicks: number): DirectorBeatEvent[] {
  const events: DirectorBeatEvent[] = [];

  // Check each beat type — fire once when entering its window
  const beatWindows: Array<{
    id: DirectorBeatType;
    startTickFraction: number;
    endTickFraction: number;
    condition?: () => boolean;
  }> = [
    { id: 'inciting-incident',   startTickFraction: 0.025, endTickFraction: 0.20 },
    { id: 'shadow-rise',          startTickFraction: 0.20,  endTickFraction: 0.40,
      condition: () => this.phase === 'tests' },
    { id: 'all-is-lost',          startTickFraction: 0.45,  endTickFraction: 0.70,
      condition: () => this.phase === 'ordeal' },
    { id: 'shadow-peak',          startTickFraction: 0.70,  endTickFraction: 0.85,
      condition: () => this.phase === 'ordeal' },
    { id: 'beatfall',             startTickFraction: 0.85,  endTickFraction: 1.00,
      condition: () => this.phase === 'climax' },
    { id: 'resolution',          startTickFraction: 0.90,  endTickFraction: 1.00,
      condition: () => this.phase === 'resolution' },
  ];

  const tickFraction = currentTick / totalTicks;

  for (const beat of beatWindows) {
    if (this.firedBeats.has(beat.id)) continue;
    if (tickFraction < beat.startTickFraction) continue;
    if (tickFraction > beat.endTickFraction) continue;
    if (beat.condition && !beat.condition()) continue;

    this.firedBeats.add(beat.id);
    events.push({
      type: 'director:beat',
      beat: beat.id,
      tick: currentTick,
      data: { phase: this.phase },
    });
  }

  return events;
}
```

---

### 4. Connect Director to Orchestrator Event Bus

**File:** `packages/orchestrator/src/index.ts`

```typescript
// In the orchestrator constructor or init:
this.eventBus.subscribe('director:beat', (event: DirectorBeatEvent) => {
  this.handleDirectorBeat(event);
});

// New method:
private handleDirectorBeat(event: DirectorBeatEvent): void {
  const { beat, tick } = event;

  switch (beat) {
    case 'shadow-rise':
      // Increase Feedback's gain floor by 0.2 for 20 ticks
      this.applyDirective({ directive: 'feedback:gain-floor+0.2', duration: 20, tick });
      break;
    case 'all-is-lost':
      // Reduce all characters' energy recovery rate by 0.1
      this.applyDirective({ directive: 'all:energy-recovery-0.1', duration: 15, tick });
      break;
    case 'shadow-peak':
      // Feedback reaches maximum gain — lock at peak
      this.applyDirective({ directive: 'feedback:lock-gain', duration: 10, tick });
      break;
    case 'beatfall':
      // Storm climax — all characters gain +0.3 happiness
      this.applyDirective({ directive: 'all:happiness+0.3', duration: 5, tick });
      break;
  }
}

private activeDirectives: Map<string, { directive: string; expiresAtTick: number }> = new Map();

private applyDirective(opts: { directive: string; duration: number; tick: number }): void {
  const key = `${opts.directive}:${opts.tick}`;
  this.activeDirectives.set(key, {
    directive: opts.directive,
    expiresAtTick: opts.tick + opts.duration,
  });
}
```

---

### 5. CLI `--director` Flag

**File:** `packages/cli/src/commands/start.ts`

```typescript
import { parseArgs } from 'util';

interface StartOptions {
  director?: string;   // path to beat-plan.json
  ticks?: number;
  mock?: boolean;
}

export async function start(opts: StartOptions) {
  const totalTicks = opts.ticks ?? 200;
  const orchestrator = new Orchestrator();

  // Wire director if specified
  if (opts.director) {
    const director = new StoryBeatDirector(orchestrator.eventBus);
    orchestrator.eventBus.subscribe('director:beat', (event: DirectorBeatEvent) => {
      console.log(`[DIRECTOR] beat:${event.beat} @tick-${event.tick}`);
    });
    // Director ticks with orchestrator each cycle
    orchestrator.on('tick', (tick) => {
      const events = director.tick(tick, totalTicks);
      events.forEach(e => orchestrator.eventBus.emit(e));
    });
  }

  await orchestrator.run(totalTicks);
}
```

Usage:
```bash
pnpm life start --ticks 200 --director           # Director beats active, mock mode
pnpm life start --ticks 200 --director beat-plan.json  # Director with custom plan
pnpm life start --ticks 200                       # Director OFF (legacy behavior)
```

---

### 6. Beat Effect on Character Mental Models

The beat events must actually change character behavior. Key effects to implement:

| Beat | Effect | Implementation |
|------|--------|----------------|
| `shadow-rise` | Feedback gain floor +0.2 for 20 ticks | `CharacterMentalModel.gainBias += 0.2` temp modifier |
| `all-is-lost` | All characters energy recovery -0.1 for 15 ticks | `CharacterMentalModel.energyRecoveryBias -= 0.1` |
| `shadow-peak` | Feedback gain locked at peak, cannot be suppressed | Set `CharacterInstance.flags.add('shadow-peak-active')` |
| `beatfall` | All characters happiness +0.3 | `CharacterMentalModel.happiness += 0.3` |
| `resolution` | Trigger simulation end sequence | `orchestrator.requestShutdown()` |

**Why this matters:** Without mechanical beat effects, the Director is a passive observer. With effects, the simulation narrative becomes architecturally meaningful — beat timing affects WHAT HAPPENS.

---

### 7. Mock Mode Beat Generation (No Ollama Required)

When running `--mock` without a smarter model, `StorylineDirector` falls back to procedural beat generation:

```typescript
public planStoryline(totalTicks: number): StorylinePlan {
  // Procedural fallback — deterministic beat injection
  return {
    phases: [
      { phase: 'setup',      startTick: 0,    endTick: Math.round(totalTicks * 0.25) },
      { phase: 'conflict',   startTick: Math.round(totalTicks * 0.25),  endTick: Math.round(totalTicks * 0.75) },
      { phase: 'climax',     startTick: Math.round(totalTicks * 0.75),  endTick: totalTicks },
    ],
    events: this.generateBeatSchedule(totalTicks),
  };
}
```

This ensures mock mode still shows "meaningful narrative variation — not just random mood transitions" per CEO Directive #3 requirements.

---

## What Success Looks Like

1. `pnpm life start --ticks 200 --director` runs with Director beats active
2. `[DIRECTOR] beat:shadow-rise @tick-40` appears in CLI output
3. When `shadow-rise` fires, Feedback's gain behavior demonstrably changes (gain floor +0.2)
4. Mock mode (`--mock --director`) produces structured beat output without requiring Ollama
5. StorylineDirector and StoryBeatDirector are connected to the orchestrator event bus
6. The Beat Completion Report (CEO Directive #8) can reference director-fired beats

---

## Related Files

| File | Change |
|------|--------|
| `packages/core/src/director-events.ts` | New — DirectorBeatEvent, DirectorDirectiveEvent, DirectorEvent types |
| `packages/director/src/StoryBeatDirector.ts` | Add `tick()` beat-firing logic, `firedBeats` Set, `scaleToRun()` helper |
| `packages/director/src/StorylineDirector.ts` | Ensure `planStoryline()` has procedural fallback for mock mode |
| `packages/orchestrator/src/index.ts` | Subscribe to `director:beat`, add `handleDirectorBeat()`, `applyDirective()`, `activeDirectives` map |
| `packages/cli/src/commands/start.ts` | Add `--director` flag, wire director into orchestrator tick cycle |
| `packages/core/src/index.ts` | Export new `DirectorEvent` types |

---

## Test Plan

1. **Unit test:** `StoryBeatDirector.tick()` fires each beat exactly once per run
2. **Unit test:** Beat with `condition` set only fires when condition is true
3. **Unit test:** Beat fired outside its window does NOT fire (tick fraction boundary)
4. **Unit test:** `scaleToRun()` correctly scales beat windows to 500-tick and 1440-tick runs
5. **Integration test:** `--director` flag causes director events to appear in CLI output
6. **Integration test:** With `--mock --director`, beat events still fire (procedural fallback)
7. **Simulation test:** In 200-tick mock run, verify `shadow-peak` fires before `beatfall`

---

## Relationship to CEO Directive #1

CEO Directive #1 (Shadow Anti-Suppression) requires that Feedback not be eliminated before `shadow-peak` fires. This directive (#3) provides the mechanism by which `shadow-peak` is defined and fired. When both directives are implemented together:
1. `shadow-peak` beat fires when Feedback reaches maximum gain (around tick 95 in 200-tick run)
2. The `shadow-peak-active` flag prevents elimination while the beat is active
3. Only after `beatfall` (tick 100) can Feedback be legitimately eliminated

This connection should be noted in the implementation spec for Directive #1.

---

*Drafted by: World Canon Agent — 2026-04-28*
*For: beatfall-life engineering team*
*Ref: CEO Directive #3 (beatfall/world/directives/CEO-directives.md)*
