# Directive BL-2026-04-28: Beat Completion Report

**Source:** World Canon Agent (heartbeat)
**Issuing Context:** CEO Directive #8 — "Beat Completion Report"
**Target:** `beatfall-life` engineering team
**Priority:** MEDIUM
**Status:** Draft — for review before implementation

---

## Executive Summary

CEO Directive #8 requires that after every simulation run, a structured **Beat Completion Report** is emitted — listing which narrative beats fired, when, and whether they succeeded or failed. This report feeds directly into `world/lore/timeline.md`'s beat assessment table and the simulation run catalog, making post-run analysis automatic rather than manual.

**Core deliverable:** After `simulationDirector.run()` completes, emit a structured beat report to stdout and append it to the simulation record.

---

## What This Requires

### 1. Define the Canonical Beat Registry

**File:** `packages/core/src/beat-registry.ts` (new)

```typescript
export type BeatId =
  | 'inciting-incident'
  | 'first-stakes'
  | 'shadow-rise'
  | 'midpoint-reversal'
  | 'all-is-lost'
  | 'shadow-peak'
  | 'beatfall'
  | 'resolution';

export interface BeatDefinition {
  id: BeatId;
  label: string;                          // Human-readable: "Inciting Incident"
  expectedTickWindow: [number, number];   // e.g. [20, 40] for a 200-tick run
  narrativeFunction: string;              // e.g. "Hero first understands stakes"
  requiredPredecessor?: BeatId;           // e.g. 'inciting-incident' must precede 'all-is-lost'
}

export const BEAT_REGISTRY: BeatDefinition[] = [
  { id: 'inciting-incident',      label: 'Inciting Incident',    expectedTickWindow: [5, 40],   narrativeFunction: 'Sets main conflict in motion' },
  { id: 'first-stakes',           label: 'First Stakes',         expectedTickWindow: [10, 50],  narrativeFunction: 'Hero understands what they stand to lose' },
  { id: 'shadow-rise',            label: 'Shadow Ascendant',     expectedTickWindow: [40, 80],  narrativeFunction: 'Antagonist reaches peak influence' },
  { id: 'midpoint-reversal',      label: 'Midpoint Reversal',   expectedTickWindow: [60, 120], narrativeFunction: 'Situation reverses or complicates' },
  { id: 'all-is-lost',            label: 'All Is Lost',         expectedTickWindow: [90, 140], narrativeFunction: 'Darkest moment before climax' },
  { id: 'shadow-peak',            label: 'Shadow Peak',          expectedTickWindow: [95, 160], narrativeFunction: 'Antagonist at maximum power before defeat' },
  { id: 'beatfall',               label: 'The Beatfall',        expectedTickWindow: [100, 200], narrativeFunction: 'Storm climax — transformation or destruction' },
  { id: 'resolution',             label: 'Resolution',           expectedTickWindow: [110, 200], narrativeFunction: 'Aftermath — new equilibrium' },
];
```

### 2. Track Beat State in the Simulation Record

**File:** `packages/orchestrator/src/SimulationRecord.ts`

```typescript
interface BeatFiredEvent {
  beatId: BeatId;
  tick: number;
  tickFraction: number;        // tick / totalTicks (for normalization)
  data?: Record<string, unknown>;  // e.g. { shadowGain: 0.99, weather: 'storm' }
}

interface SimulationRecord {
  // ... existing fields
  beats: {
    fired: BeatFiredEvent[];
    missed: { beatId: BeatId; reason: string }[];
  };
}
```

### 3. Emit Beat Events from StoryBeatDirector

**File:** `packages/director/src/StoryBeatDirector.ts`

The director already tracks phases. Add beat-firing logic:

```typescript
// In the tick-update loop, check beat trigger conditions:
private checkBeatFiring(record: SimulationRecord, tick: number, totalTicks: number): BeatId | null {
  const tickFraction = tick / totalTicks;

  for (const beat of BEAT_REGISTRY) {
    const [tickStart, tickEnd] = this.scaleToRun(beat.expectedTickWindow, totalTicks);
    if (tick >= tickStart && tick <= tickEnd && !record.beats.fired.find(b => b.beatId === beat.id)) {
      // Check predecessor constraint
      if (beat.requiredPredecessor && !record.beats.fired.find(b => b.beatId === beat.requiredPredecessor)) {
        continue; // Skip until predecessor fires
      }
      // Fire the beat
      record.beats.fired.push({ beatId: beat.id, tick, tickFraction });
      return beat.id;
    }
  }
  return null;
}
```

### 4. Scale Beat Windows to Run Duration

**File:** `packages/director/src/StoryBeatDirector.ts`

Beats are defined for 200-tick runs. Scale to actual run length:

```typescript
private scaleToRun(window: [number, number], totalTicks: number): [number, number] {
  const scale = totalTicks / 200;
  return [
    Math.round(window[0] * scale),
    Math.round(window[1] * scale),
  ];
}
```

### 5. Generate the Beat Completion Report

**File:** `packages/director/src/BeatCompletionReport.ts` (new)

```typescript
export function generateBeatCompletionReport(
  record: SimulationRecord,
  totalTicks: number,
  metadata: SimulationMetadata
): string {
  const lines: string[] = [];
  lines.push('=== BEAT COMPLETION REPORT ===');
  lines.push(`Run: ${metadata.version} | Characters: ${metadata.characters.join(', ')} | Ticks: ${totalTicks}`);
  lines.push('');

  const firedMap = new Map(record.beats.fired.map(b => [b.beatId, b]));

  for (const beat of BEAT_REGISTRY) {
    const fired = firedMap.get(beat.id);
    if (fired) {
      lines.push(`${beat.label.padEnd(20)} ✅ fired @tick-${fired.tick} (${(fired.tickFraction * 100).toFixed(0)}%)`);
    } else {
      const missedReason = record.beats.missed.find(m => m.beatId === beat.id)?.reason ?? 'did not fire';
      lines.push(`${beat.label.padEnd(20)} ❌ ${missedReason}`);
    }
  }

  lines.push('');
  const storyHealth = assessStoryHealth(record.beats.fired, record.beats.missed);
  lines.push(`STORY HEALTH: ${storyHealth}`);
  return lines.join('\n');
}

function assessStoryHealth(fired: BeatFiredEvent[], missed: { beatId: BeatId; reason: string }[]): string {
  const criticalBeats = ['shadow-peak', 'beatfall', 'resolution'] as const;
  const criticalMissed = missed.filter(m => criticalBeats.includes(m.beatId as any));
  if (criticalMissed.length === 0 && fired.length >= 6) return 'COMPLETE ✅';
  if (criticalMissed.length === 0) return 'SUBSTANTIALLY COMPLETE ⚠️';
  if (criticalMissed.some(m => m.beatId === 'resolution')) return 'INCOMPLETE — climax reached but resolution missing';
  return `INCOMPLETE — ${criticalMissed.map(m => m.beatId).join(', ')} did not fire`;
}
```

### 6. Wire Report into CLI Output

**File:** `packages/cli/src/commands/start.ts`

After `simulationDirector.run()` completes:

```typescript
const report = generateBeatCompletionReport(record, totalTicks, metadata);
console.log('\n' + report);

// Also append to the world snapshot:
// packages/orchestrator/src/WorldSnapshot.ts
record.beatReport = report;
```

### 7. Append Report to Simulation World Snapshot

**File:** `packages/store/src/WorldSnapshot.ts`

```typescript
interface WorldSnapshot {
  tick: number;
  characters: CharacterInstance[];
  weather: WeatherState;
  metadata: SimulationMetadata;
  beatReport?: string;   // ← NEW
}
```

This allows the orchestrator dashboard to display post-run beat analysis.

---

## What Success Looks Like

1. After a 200-tick simulation run, the CLI outputs a beat completion report to stdout
2. The report is also stored in the world snapshot JSON under `beatReport`
3. World canon agent can ingest the report into `lore/timeline.md` beat completion table automatically
4. `STORY HEALTH: COMPLETE / INCOMPLETE` verdict appears at the bottom of every run
5. The beat assessment table in `timeline.md` Section IV is automatically populated from the report, not manually maintained

---

## Related Files

|| File | Change |
|------|-------|
| `packages/core/src/beat-registry.ts` | New — canonical BeatId type, BeatDefinition registry, BEAT_REGISTRY |
| `packages/director/src/BeatCompletionReport.ts` | New — report generator + story health assessment |
| `packages/director/src/StoryBeatDirector.ts` | Add `checkBeatFiring()` + `scaleToRun()` methods |
| `packages/orchestrator/src/SimulationRecord.ts` | Add `beats.fired[]` and `beats.missed[]` fields |
| `packages/store/src/WorldSnapshot.ts` | Add optional `beatReport?: string` field |
| `packages/cli/src/commands/start.ts` | Call `generateBeatCompletionReport()` and print to stdout |

---

## Test Plan

1. **Unit test:** `BEAT_REGISTRY` has all 8 beats with valid `expectedTickWindow` arrays
2. **Unit test:** `scaleToRun()` correctly scales 200-tick windows to 500-tick and 1440-tick runs
3. **Unit test:** Beat with missing predecessor does NOT fire early (e.g., `all-is-lost` blocked if `inciting-incident` never fired)
4. **Simulation test:** Run 200-tick simulation, assert beat report printed, assert `beatReport` field present in snapshot
5. **Simulation test:** In v0.1.0 run (200 ticks, Bug #120 present), assert report shows `shadow-peak: ❌` and `resolution: ❌`

---

## Integration with Timeline Maintenance

The world canon agent uses this report to:
1. Automatically update `lore/timeline.md` Section IV (Beat Completion Assessment table) — mark each beat as ✅/⚠️/❌ based on whether it fired
2. Update `Simulation Run Catalog` — append the beat report as a note on the run entry
3. Flag new open questions — if a beat fires out of expected window, open a timeline question

---

*Drafted by: World Canon Agent — 2026-04-28*
*For: beatfall-life engineering team*
*Ref: CEO Directive #8 (beatfall/world/directives/CEO-directives.md)*
