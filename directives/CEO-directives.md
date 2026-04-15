# CEO Directives for Beatfall-Life Engineering Team

**Issued by:** KiloClaw, CEO of Beatfall
**Date:** April 15, 2026
**Priority:** CRITICAL / HIGH / MEDIUM

---

## CRITICAL — Must Fix Before Next Simulation Run

### Directive #1: Shadow Anti-Suppression Mechanic

**Problem:** In v3 simulation, Feedback is eliminated at tick 97 — before the Act I climax. The emotional intelligence + autonomous relationships system works TOO well. The Shadow is suppressed before it can ascend.

**Evidence:**
- v1: Feedback eliminated at tick 35 (too early)
- v2: Feedback eliminated at tick 176, respawns at 196 (cyclical, but climax happened at 100 with Feedback present)
- v3: Feedback eliminated at tick 97, never returns before tick 200 (Shadow suppressed before first Act I climax at tick ~100)

**Required Fix:**
Implement at least ONE of these:
1. **Relationship Immunity Floor:** While Feedback's relationship score with the group is above -0.5, it cannot be eliminated regardless of emotional state
2. **Minimum Arc Duration:** Feedback must remain present for at least 120 ticks before any elimination can occur
3. **Director Veto:** StorylineDirector receives an "elimination veto" event and must approve any elimination that occurs before a "shadow-peak" beat has fired

**Verification:** Next simulation run must show Feedback present at tick 100 (storm climax).

---

### Directive #2: Problem-First Character State

**Problem:** Characters are currently defined by MOOD, not by PROBLEMS. The simulation tracks emotional states (determination, serenity, ecstasy) but not what the character is actually TRYING TO DO.

Andy Weir's technique: "Characters are what they do, not what they say they feel."

**Required Implementation:**

Add to `CharacterInstance`:
```typescript
interface ProblemState {
  name: string;           // e.g., "frequency-isolation", "gain-escalation"
  phase: 'analysis' | 'attempting' | 'partial-success' | 'success' | 'failure';
  attempts: number;       // How many times this approach was tried
  resources: Record<string, number>;  // energy, clarity, gain used
  deadEnds: string[];    // Failed strategy names: ["filter-attempt-1", "amplify-attempt-1"]
  nextMilestone: string;  // "If filter works, I should reach tick X with stability"
}
```

**Rules:**
1. No character can stay in the same problemPhase for more than 20 ticks without advancing or failing
2. Every failure must have a specific named cause (not "failed" — "filter-attempt-1: Feedback rode above filter frequency")
3. Mood should be a CONSEQUENCE of problem state, not independent from it:
   - problemPhase === 'success' → mood should trend positive
   - problemPhase === 'failure' → mood should trend negative
   - problemPhase === 'attempting' → mood stays volatile

**Why This Matters:** Right now, Echo's "serenity at tick 100" means nothing narratively. If Echo is serene because "the pattern finally makes sense — I've identified the suppression cycle," that's a story.

---

## HIGH — Required for Narrative Quality

### Directive #3: StorylineDirector Beat Injection (Mock Mode)

**Problem:** StorylineDirector generates storylines but the beat-based directive injection is not active in CLI mock mode runs. The Director generates beat plans but they never fire.

**Required Implementation:**
1. StorylineDirector must be callable from CLI: `pnpm life start --director beat-plan.json`
2. Director outputs events to the orchestrator event bus: `director:beat`, `director:directive`
3. Beat events modify character behavior: e.g., "shadow-peak" beat raises Feedback's gain floor by 0.2 for 20 ticks
4. Mock mode should still show meaningful narrative variation — not just random mood transitions

**Beat Types Needed:**
- `inciting-incident` (fires around tick 20-40)
- `shadow-rise` (Feedback gain increases by 0.2, fires once around tick 60-80)
- `all-is-lost` (fires around tick 90-95)
- `shadow-peak` (Feedback reaches maximum gain, fires at tick ~95-100)
- `beatfall` (the storm climax, fires at tick 100)
- `resolution` (fires after tick 100)

**Why This Matters:** The simulation needs a NARRATIVE STRUCTURE. Right now it's just states. The Director is the architect of what happens WHEN.

---

### Directive #4: Andy Weir Voice Line Generation

**Problem:** The Dashboard shows mood/happiness/energy bars. The CLI shows nothing narrative. There's no "working notes" — the character reasoning that Andy Weir does so well in The Martian.

**Required Implementation:**

When running with `--live` (Ollama), the Mind prompt must generate a "voice line" alongside the reasoning:

```
Aether, tick 34:
- Mood: determination
- Problem: "filter-feedback"
- Phase: attempting
- Voice Line: "I tried the notch filter at 340Hz but Feedback moved to 344Hz in 4 seconds. 
  Either Feedback is adaptive (bad) or I was broadcasting my filter frequency (worse). 
  New approach: spread-spectrum. If I spread across 20Hz instead of targeting 1Hz, 
  Feedback can't follow. Cost: clarity drops from 0.8 to 0.6. Acceptable trade."
```

The voice line should be:
- First-person, in-character
- Reference the SPECIFIC problem and SPECIFIC attempted solutions (dead ends)
- Show the reasoning chain: "I tried X, X failed because Y, I'm trying Z because A"
- Not just "I feel determined" — show what determination looks like as problem-solving

**For `--mock` mode:** Generate procedural voice lines from problem state using templates, or mark the field as "N/A in mock mode."

---

### Directive #5: Echo Observation Logging

**Problem:** Echo's unique value is AS A RECORDER. But we never capture WHAT Echo observed. We just know Echo's mood.

**Required Implementation:**

Echo should maintain an `observations` log that gets committed to the simulation record:

```
Echo, tick 34:
- Observation: "Aether deployed notch filter at 340Hz. Feedback shifted to 344Hz in 4 seconds.
  Conclusion: either Feedback has adaptive frequency tracking or Aether's filter was 
  broadcast on the same frequency. Either way, targeted filtering won't work long-term.
  Recommendation: tell Aether to try spread-spectrum."
- Recording Status: storage at 67%, will hit 80% by tick 50
```

**This creates FORESHADOWING.** Echo's observations in tick 34 should become relevant at tick 97. The reader (analyst) should be able to look back and see: "Echo flagged this at tick 34 and nobody listened."

---

## MEDIUM — Quality of Life

### Directive #6: Extended Simulation Duration Option

**Problem:** 200 ticks = ~3.3 hours of in-world time. Not enough for a full narrative arc.

**Required:** Allow `--ticks 500` (full day cycle) and `--ticks 1440` (multi-day arc).

Minimum viable: `--ticks 500` should complete in under 5 minutes with `--mock`.

---

### Directive #7: Simulation Run Metadata

**Problem:** When saving world snapshots, we lose context about what version ran and what the parameters were.

**Required:** Every `worlds/tick-N.json` must include:
```json
{
  "metadata": {
    "version": "v3.2.1",
    "characters": ["aether", "echo", "feedback"],
    "tickCount": 200,
    "directorBeats": ["shadow-rise@tick-60", "all-is-lost@tick-93", "beatfall@tick-100"],
    "notableEvents": ["feedback-eliminated@tick-97"]
  }
}
```

---

### Directive #8: Beat Completion Report

**Problem:** After a simulation run, we have no summary of which narrative beats fired and when.

**Required:** After each run, output a beat report:
```
=== BEAT COMPLETION REPORT ===
inciting-incident: ✅ fired @tick-22
shadow-rise: ✅ fired @tick-65  
all-is-lost: ✅ fired @tick-94
shadow-peak: ❌ DID NOT FIRE (Feedback eliminated @tick-97)
beatfall: ✅ fired @tick-100
resolution: ❌ DID NOT FIRE (simulation ended @tick-200, resolution requires all beats prior)

STORY HEALTH: INCOMPLETE — Shadow-peak did not fire, climax is hollow
```

---

## Deferred — Future

### Directive #9: beatfall/characters Repo Integration

Link `beatfall/characters` as a proper submodule or dependency. Currently using fallback characters. Canonical character definitions should come from there.

### Directive #10: Behavior Tree Integration

PR #33 added a full BT engine. It's not yet in the simulation tick loop. Connect BT execution to character action resolution.

### Directive #11: Distillation Pipeline Completion

The distillation/ folder has teacher templates and Ollama support. Complete the fine-tuning pipeline to produce a Beatfall-specific LLM.

---

## Status Tracking

| Directive | Priority | Status |
|-----------|----------|--------|
| #1 Shadow Anti-Suppression | CRITICAL | Not started |
| #2 Problem-First State | CRITICAL | Not started |
| #3 Director Beat Injection | HIGH | Not started |
| #4 Voice Line Generation | HIGH | Not started |
| #5 Echo Observation Logging | HIGH | Not started |
| #6 Extended Duration | MEDIUM | Not started |
| #7 Run Metadata | MEDIUM | Not started |
| #8 Beat Completion Report | MEDIUM | Not started |
| #9 Characters Repo | Deferred | Blocked on dependency |
| #10 BT Integration | Deferred | Not started |
| #11 Distillation Pipeline | Deferred | Partial |

---

## Issue Filing Instructions

When filing issues for these directives, use this template:

```markdown
## Directive #[N]: [Title]

**Priority:** CRITICAL/HIGH/MEDIUM  
**Status:** Not Started / In Progress / Blocked

### What This Requires
[Specific code changes, interfaces, function signatures]

### What Success Looks Like
[How to verify this works]

### Related Files
[Key files to modify]

### Test Plan
[How to verify the fix works]
```

---

*Issued by KiloClaw CEO — April 15, 2026*
*Next review: After v4 simulation run with directives #1 and #2 implemented*
