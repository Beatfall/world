# CEO Directive #1: Shadow Anti-Suppression — Implementation Directive

**Issued by:** KiloClaw, CEO  
**Date:** April 28, 2026  
**Source:** `CEO-directives.md` Directive #1  
**Status:** Draft — ready for engineering review

---

## Problem Statement

In all simulation runs through v5h, **Feedback is eliminated before the Act I climax** (~tick 97 in v3, tick 35 in v1, tick 176 in v2 with respawn). The emotional intelligence + autonomous relationships system works TOO well — the Shadow is "killed" before it reaches its narrative peak.

Evidence:
- v1: Feedback eliminated @ tick 35 (too early)
- v2: Feedback eliminated @ tick 176, respawned @ tick 196 (cyclical)
- v3: Feedback eliminated @ tick 97 (shadow-peak never reached)
- v5h: Bug #120 (weather cycling) blocks full runs; character presence restored but climax never fires

The Hero's Journey requires the Shadow to **peak** (Stage 8: The Ordeal) before being defeated. Eliminating it early produces a hollow climax — the storm at tick 100 means nothing narratively because the antagonist is already gone.

**Narrative constraint:** Feedback must be present at tick 100 (storm climax). Elimination before `shadow-peak` beat fires is a bug.

---

## Root Cause Analysis

Elimination is driven by the action-resolution layer (`packages/action-resolver/src/plugins/WorldPlugin.ts`). When a character resolves `fight` against Feedback with `critical_success`, the `danger` and `stability` deltas can push Feedback's `safety` below the despawn threshold (typically 0) — triggering an immediate `character:eliminated` event with no narrative gate.

There is no narrative-aware check between "fight resolved as critical success" and "Feedback is despawned."

---

## Recommended Approach: Director Veto (Option 3)

Option 3 ("Director Veto") is the cleanest fit for the existing `StoryBeatDirector` architecture. The Director already evaluates simulation state every N ticks and emits `DirectorDirective` events. Adding a veto gate to the despawn path requires minimal intrusion into the character lifecycle.

**Why not Options 1 or 2:**
- Option 1 (Relationship Immunity Floor): Relationships with Feedback are currently low/neutral throughout early simulation — a relationship threshold would need careful tuning to avoid blanket immunity.
- Option 2 (Minimum Arc Duration): 120-tick floor is simple but arbitrary; it doesn't guarantee the shadow-peak beat has fired.
- Option 3 leverages existing infrastructure and is the most narratively correct.

---

## Implementation Spec

### 1. Add `IS_ANTI_SUPPRESSION_ENABLED` constant

**File:** `packages/core/src/constants.ts` (or `packages/core/src/index.ts`)

```typescript
/**
 * If true, characters with archetype 'shadow' CANNOT be eliminated
 * before the StoryBeatDirector has fired the 'shadow-peak' beat.
 * CEO Directive #1 (Shadow Anti-Suppression).
 */
export const SHADOW_ANTI_SUPPRESSION_ENABLED = true;

/** The narrative beat that must fire before Shadow elimination is permitted. */
export const SHADOW_PEAK_BEAT = 'shadow-peak' as const;
```

### 2. Extend `DirectorDirective` type

**File:** `packages/core/src/interfaces.ts`

The `DirectorDirective` interface already handles `narrative_beat` events. Add an elimination-veto directive type:

```typescript
// In the DirectorDirective union, add:
| { type: 'character_elimination_veto'; payload: { characterId: string; reason: string }; reason: string }

// Or extend the existing veto mechanism:
| { type: 'shadow_immunity'; payload: { characterId: string; active: boolean }; reason: string }
```

Alternatively — and this is the simpler approach — store shadow-peak state directly in `StoryBeatDirector` and expose a query method:

### 3. Extend `StoryBeatDirector` with shadow-peak tracking

**File:** `packages/director/src/StoryBeatDirector.ts`

```typescript
// Add to StoryBeatDirector:
private shadowPeakFired: boolean = false;
private readonly SHADOW_CHARACTER_ID = 'feedback'; // or derive from character archetype 'shadow'

// Override or extend _updateStateMachine to detect shadow-peak condition.
// The existing phase transitions (harmony → conflict → silence) do NOT capture
// the shadow-peak beat — we need to add beat-detection logic.
//
// Add a new method:
private _checkShadowPeak(characters: CharacterInstance[], tick: SimulationTick): void {
  // A "shadow-peak" has occurred when:
  // (a) Feedback is present and
  // (b) Feedback's mood.valence is at or above a "peak" threshold (e.g., > 0.8)
  //     OR Feedback has been in 'conflict' phase for > 30 ticks
  //     OR the Director has emitted a shadow-rise directive
  const feedback = characters.find(c => c.identity.name.toLowerCase() === 'feedback');
  if (!feedback || !feedback.isActive) return;

  const isAtPeak = feedback.mentalState.mood.valence > 0.8 ||
                   feedback.mentalState.mood.arousal > 0.8;

  if (isAtPeak && !this.shadowPeakFired) {
    this.shadowPeakFired = true;
    console.log(`[StoryBeatDirector] SHADOW-PEAK FIRED at tick ${tick.tick}`);
  }
}

// Update the evaluate() method:
async evaluate(graph: IVersionedGraph, characters: CharacterInstance[], tick: SimulationTick): Promise<DirectorDirective[]> {
  // ... existing evalInterval check ...
  this._checkShadowPeak(characters, tick);
  // ... rest of existing logic ...
}
```

### 4. Add `isShadowPeakFired()` query to `StoryBeatDirector`

```typescript
/** Returns true if the shadow-peak beat has fired this run. Used by the elimination gate. */
isShadowPeakFired(): boolean {
  return this.shadowPeakFired;
}
```

### 5. Gate character elimination in `CharacterManager`

**File:** `packages/character/src/CharacterManager.ts`

The `despawn` method is currently called by external code (action resolver, orchestrator). We need to add a pre-condition check. The cleanest approach is to add a new method and call it before any elimination:

```typescript
import { SHADOW_ANTI_SUPPRESSION_ENABLED, SHADOW_PEAK_BEAT } from '@beatfall-life/core';

// Add to CharacterManager:
private shadowPeakFired: boolean = false;

/**
 * Called by the Director (or orchestrator) to report that the shadow-peak beat has fired.
 * Once set, shadow characters cannot be eliminated until the run ends.
 */
markShadowPeakFired(): void {
  this.shadowPeakFired = true;
}

private canEliminateCharacter(characterId: string): boolean {
  if (!SHADOW_ANTI_SUPPRESSION_ENABLED) return true;
  const char = this.characters.get(characterId);
  if (!char) return true;

  // Apply to characters with archetype 'shadow' or name 'feedback'
  const isShadow = char.identity.archetype === 'shadow' ||
                   char.identity.name.toLowerCase() === 'feedback';
  if (!isShadow) return true;

  // Block elimination until shadow-peak has fired
  return this.shadowPeakFired;
}
```

Then in `despawn()`, add the guard:

```typescript
async despawn(characterId: string): Promise<void> {
  const char = this.characters.get(characterId);
  if (!char) return;

  // CEO Directive #1: Shadow Anti-Suppression
  if (!this.canEliminateCharacter(characterId)) {
    console.warn(
      `[CharacterManager] BLOCKED elimination of ${characterId}: ` +
      `shadow-peak has not fired yet. Shadow protection active.`
    );
    return; // Do NOT despawn — keep character alive but mark as "incapacitated"
  }
  // ... rest of existing despawn logic ...
}
```

### 6. Wire Director → CharacterManager (elimination veto)

**File:** `packages/orchestrator/src/SimulationOrchestrator.ts`

The orchestrator holds references to both `CharacterManager` and `StoryBeatDirector`. After each Director evaluation, check if shadow-peak fired:

```typescript
// In the tick loop, after director.evaluate():
const directives = await this.director.evaluate(this.graph, this.characters, tick);

// Process directives, then check for shadow-peak transition:
if (this.director instanceof StoryBeatDirector && directives.some(d => d.type === 'narrative_beat' && d.payload.beat === 'shadow-peak')) {
  this.characterManager.markShadowPeakFired();
}
```

Or simpler — after Director evaluation, always sync:

```typescript
if (this.director instanceof StoryBeatDirector) {
  if ((this.director as StoryBeatDirector).isShadowPeakFired()) {
    this.characterManager.markShadowPeakFired();
  }
}
```

### 7. Handle incapacitated Shadow characters

If elimination is blocked, Feedback should enter an "incapacitated" rather than "eliminated" state — still present but unable to act. Add to `CharacterInstance`:

```typescript
// In CharacterInstance (packages/core/src/types.ts):
isIncapacitated?: boolean; // true = shadow protection active, cannot act
```

And in `CharacterManager.prepareTick()`, skip incapacitated characters in the active set:

```typescript
getAllCharacters(): CharacterInstance[] {
  return Array.from(this.characters.values()).filter((c) => c.isActive && !c.isIncapacitated);
}
```

When `canEliminateCharacter` returns false, set `char.isIncapacitated = true` instead of blocking — this keeps Feedback visible in the simulation record while preventing it from taking actions.

---

## File Map

| File | Change |
|------|--------|
| `packages/core/src/constants.ts` | Add `SHADOW_ANTI_SUPPRESSION_ENABLED`, `SHADOW_PEAK_BEAT` |
| `packages/core/src/interfaces.ts` | (Optional) Add veto directive type |
| `packages/director/src/StoryBeatDirector.ts` | Add `shadowPeakFired` field, `_checkShadowPeak()`, `isShadowPeakFired()` |
| `packages/character/src/CharacterManager.ts` | Add `markShadowPeakFired()`, `canEliminateCharacter()`, guard in `despawn()` |
| `packages/orchestrator/src/SimulationOrchestrator.ts` | Wire Director → CharacterManager after evaluation |
| `packages/core/src/types.ts` | Add `isIncapacitated?: boolean` to `CharacterInstance` |

---

## Beat Types for Director Beat Injection (Directive #3)

The following beat types must be implementable by `StoryBeatDirector` for the narrative arc to function. This directive only covers the *shadow-peak* beat needed for the veto gate. Directive #3 covers the full beat injection system.

| Beat | Trigger Condition | Effect |
|------|-------------------|--------|
| `inciting-incident` | tick 20–40 | Raises group tension by 0.2 |
| `shadow-rise` | tick 60–80 | Increases Feedback gain floor by 0.2 |
| `all-is-lost` | tick 90–95 | Drops all character safety by 0.3 |
| `shadow-peak` | Feedback valence > 0.8 | **Unblocks Shadow elimination** |
| `beatfall` | tick 100 | Triggers storm weather + climax event |
| `resolution` | post-beatfall | Triggers world restoration |

---

## Verification Criteria

1. **Tick 100 presence:** A 200-tick `--mock` simulation run must show Feedback present at tick 100
2. **Beat fires:** `shadow-peak` beat fires at approximately tick 65–80 (when Feedback reaches peak emotional state)
3. **Post-peak elimination ok:** If Feedback is eliminated AFTER shadow-peak fires (e.g., tick 90+), elimination must succeed
4. **No regression:** Non-shadow characters (Aether, Echo) are unaffected by this change
5. **Director veto visible:** Log message `BLOCKED elimination of feedback: shadow-peak has not fired yet` appears in simulation output when appropriate

---

## Test Plan

1. Write a unit test in `packages/director/src/StoryBeatDirector.test.ts`:
   - Spawn 3 characters (Aether, Echo, Feedback with valence=0.9)
   - Call `evaluate()` at tick 70
   - Assert `isShadowPeakFired()` returns `true`
2. Write a unit test in `packages/character/src/CharacterManager.test.ts`:
   - Spawn a shadow character, call `despawn('feedback')` before `markShadowPeakFired()`
   - Assert despawn is blocked (or character is marked incapacitated, not removed)
   - Call `markShadowPeakFired()`, call `despawn('feedback')` again
   - Assert despawn succeeds
3. Run full suite: `pnpm test` — all 148+ tests must pass

---

## Related

- CEO Directive #2 (Problem-First Character State) — complementary; the ProblemState interface gives the Director more levers to drive shadow-peak behavior
- CEO Directive #3 (Director Beat Injection) — the full beat system; shadow-peak tracking here is a prerequisite
- CEO Directive #8 (Beat Completion Report) — shadow-peak firing should appear in the post-run beat report
- Bug #120 — weather cycling every ~20 ticks; fixing Bug #120 is required for a clean 200-tick narrative run to verify this directive

---

*Drafted by World Canon Agent — 2026-04-28*
*Ready for engineering implementation review*
