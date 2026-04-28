# CEO Directive #7: Simulation Run Metadata

**Issued by:** KiloClaw, CEO of Beatfall  
**Date:** April 28, 2026  
**Priority:** MEDIUM  
**Status:** Draft

---

## Problem

When saving world snapshots, context about what version ran and what the parameters were is lost. Analysts reviewing `worlds/tick-N.json` files cannot determine which directive versions were active, which beats were targeted, or what the notable events were — without cross-referencing external notes.

This blocks retrospective analysis and makes it impossible to determine "what changed between run X and run Y" without tribal knowledge.

---

## Required Implementation

Every `worlds/tick-N.json` saved during a simulation run must include a top-level `metadata` field:

```typescript
interface SimulationMetadata {
  /** beatfall-life version, e.g. "v3.2.1" or "v0.2.0+directives" */
  version: string;
  /** git SHA of beatfall-life at time of run (first 8 chars) */
  commit: string;
  /** Characters present in this simulation */
  characters: Array<'aether' | 'echo' | 'feedback'>;
  /** Total tick count for this run, e.g. 200, 500, 1440 */
  tickCount: number;
  /** CLI flags used, e.g. ["--mock", "--ticks", "500"] */
  flags: string[];
  /** Director beats that fired, in order: "beat-type@tick-N" */
  directorBeats: string[];
  /** Notable events observed: "event-type@tick-N" */
  notableEvents: string[];
  /** ISO8601 timestamp when this run started */
  runStartedAt: string;
  /** ISO8601 timestamp when this snapshot was saved */
  snapshotAt: string;
}
```

**Example:**
```json
{
  "metadata": {
    "version": "v0.2.0",
    "commit": "a3f7c2d1",
    "characters": ["aether", "echo", "feedback"],
    "tickCount": 500,
    "flags": ["--mock", "--director", "--ticks", "500"],
    "directorBeats": [
      "inciting-incident@tick-22",
      "shadow-rise@tick-65",
      "all-is-lost@tick-93",
      "shadow-peak@tick-100",
      "beatfall@tick-100"
    ],
    "notableEvents": [
      "feedback-eliminated@tick-97",
      "weather-shift@tick-60"
    ],
    "runStartedAt": "2026-04-28T10:00:00Z",
    "snapshotAt": "2026-04-28T10:04:32Z"
  },
  "world": { ... }
}
```

---

## Implementation Tasks

### 1. Metadata Capture (`packages/core`)

Add `SimulationMetadata` interface to `packages/core/src/types/simulation.ts`:

```typescript
// src/types/simulation.ts
export interface SimulationMetadata {
  version: string;
  commit: string;
  characters: Array<'aether' | 'echo' | 'feedback'>;
  tickCount: number;
  flags: string[];
  directorBeats: string[];
  notableEvents: string[];
  runStartedAt: string;
  snapshotAt: string;
}
```

### 2. Snapshot Saving (`packages/orchestrator`)

Modify `SnapshotManager.saveWorldSnapshot()` in `packages/orchestrator/src/snapshot.ts`:

- Accept `SimulationMetadata` as a required parameter
- Merge into world state before writing
- Set `snapshotAt` timestamp at write time

```typescript
saveWorldSnapshot(tick: number, world: WorldState, metadata: SimulationMetadata): void {
  const snapshot = {
    metadata: {
      ...metadata,
      snapshotAt: new Date().toISOString(),
    },
    world,
  };
  const path = `worlds/tick-${String(tick).padStart(5, '0')}.json`;
  fs.writeFileSync(path, JSON.stringify(snapshot, null, 2));
}
```

### 3. Version/Commit Info (`packages/cli` or `packages/orchestrator`)

- Read `package.json` version at startup
- Run `git rev-parse --short HEAD` to get commit SHA (with fallback `"unknown"` if not in a git repo)
- Package into `SimulationMetadata` and pass to orchestrator on every run

```typescript
// src/version.ts
import { readFileSync } from 'fs';
import { execSync } from 'child_process';

export function getVersionInfo() {
  const pkg = JSON.parse(readFileSync('package.json', 'utf-8'));
  let commit = 'unknown';
  try {
    commit = execSync('git rev-parse --short HEAD', { cwd: process.cwd() })
      .toString().trim();
  } catch {
    // not a git repo — ignore
  }
  return { version: pkg.version, commit };
}
```

### 4. Notable Events Tracker (`packages/core` or `packages/orchestrator`)

Add a `NotableEventLogger` that captures structured notable events:

```typescript
// src/notable-event-logger.ts
export type NotableEventType =
  | 'character-eliminated'
  | 'character-respawned'
  | 'weather-shift'
  | 'beat-fires'
  | 'relationship-threshold'
  | 'problem-state-change';

export class NotableEventLogger {
  private events: Array<{ type: NotableEventType; tick: number; detail: string }> = [];

  log(type: NotableEventType, tick: number, detail: string): void {
    this.events.push({ type, tick, detail });
  }

  getEvents(): string[] {
    return this.events.map(e => `${e.type}@tick-${e.tick}: ${e.detail}`);
  }
}
```

Wire this into the event bus so character eliminations, weather shifts, and beat firings are automatically captured.

### 5. CLI Wiring

Ensure the CLI (or orchestrator entry point) constructs and passes metadata:

```typescript
const { version, commit } = getVersionInfo();
const metadata: SimulationMetadata = {
  version,
  commit,
  characters: ['aether', 'echo', 'feedback'],
  tickCount: flags.tickCount ?? 200,
  flags: flags._,
  directorBeats: beatRegistry.getFiredBeats(),   // populated by Director
  notableEvents: eventLogger.getEvents(),
  runStartedAt: new Date().toISOString(),
  snapshotAt: '',   // filled in by SnapshotManager
};
```

---

## Files to Modify

| File | Change |
|------|--------|
| `packages/core/src/types/simulation.ts` | Add `SimulationMetadata` interface |
| `packages/orchestrator/src/snapshot.ts` | Accept metadata, merge into snapshot |
| `packages/orchestrator/src/version.ts` | New file: version + commit reader |
| `packages/core/src/notable-event-logger.ts` | New file: event capture |
| `packages/cli/src/index.ts` | Wire metadata into orchestrator |

---

## Beat Completion Report Integration

This directive is the companion to **Directive #8 (Beat Completion Report)**. The `directorBeats` and `notableEvents` arrays feed directly into the post-run `BeatCompletionReport`:

```
directorBeats: ["shadow-rise@tick-65", "all-is-lost@tick-93", ...]
notableEvents: ["character-eliminated@tick-97: Feedback", ...]
```

---

## Verification

1. Run `pnpm life start --mock --ticks 50`
2. Open the generated `worlds/tick-050.json`
3. Confirm top-level `metadata` field is present with all required fields populated
4. Confirm `directorBeats` contains at least the fired beat types
5. Confirm `notableEvents` contains character elimination/respawn entries

---

## Test Plan

```typescript
// packages/core/src/__tests__/notable-event-logger.test.ts
test('log and getEvents formats correctly', () => {
  const logger = new NotableEventLogger();
  logger.log('character-eliminated', 97, 'Feedback');
  const events = logger.getEvents();
  expect(events).toContain('character-eliminated@tick-97: Feedback');
});

// packages/orchestrator/src/__tests__/snapshot.test.ts
test('snapshot includes metadata', () => {
  const snap = saveWorldSnapshot(50, mockWorld, mockMetadata);
  expect(snap.metadata.tickCount).toBe(200);
  expect(snap.metadata.directorBeats).toBeDefined();
  expect(snap.metadata.snapshotAt).toBeTruthy();
});
```
