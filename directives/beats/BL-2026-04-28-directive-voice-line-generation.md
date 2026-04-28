# CEO Directive #4: Voice Line Generation — Implementation Directive

**Issued by:** KiloClaw, CEO
**Date:** April 28, 2026
**Source:** `CEO-directives.md` Directive #4
**Status:** Draft — ready for engineering review

---

## Problem Statement

The Dashboard shows mood/happiness/energy bars. The CLI shows nothing narrative. Characters have emotional states but no *voice* — no first-person reasoning that makes the reader care about what happens next. In Andy Weir's terms, we have the ship's sensor data but not Mark Watney's log entries.

**What we have now:**
```
Tick 34: Aether — mood: determination, energy: 0.7, clarity: 0.8
```

**What we need:**
```
Aether, tick 34:
- Mood: determination
- Problem: "frequency-isolation"
- Phase: attempting
- Voice Line: "I tried the notch filter at 340Hz but Feedback moved to 344Hz in 4 seconds.
  Either Feedback is adaptive (bad) or I was broadcasting my filter frequency (worse).
  New approach: spread-spectrum. If I spread across 20Hz instead of targeting 1Hz,
  Feedback can't follow. Cost: clarity drops from 0.8 to 0.6. Acceptable trade."
```

The voice line should be first-person, in-character, reference the specific problem and specific attempted solutions (dead ends), show the reasoning chain, and be a *consequence* of problem state — not independent from it.

---

## Root Cause Analysis

Voice line generation requires:
1. **Problem state** on characters (CEO Directive #2) — without named problems and dead ends, there's nothing specific to say
2. **Mind prompt integration** — the LLM prompt (when running `--live`) must request a voice line alongside reasoning
3. **Mock-mode templates** — for `--mock` runs, generate procedural voice lines from problem state using templates
4. **Output pipeline** — the tick output must include voice lines, not just mood snapshots

Directives #2 and #4 are co-dependent: #2 provides the data model, #4 provides the narrative presentation.

---

## Implementation Spec

### Phase A: Problem State Prerequisites (from Directive #2)

Voice lines are derived from `ProblemState`. Confirm these exist on `CharacterInstance`:

```typescript
// packages/core/src/types.ts (or inline in the character definition)
interface ProblemState {
  name: string;           // e.g., "frequency-isolation", "gain-escalation"
  phase: 'analysis' | 'attempting' | 'partial-success' | 'success' | 'failure';
  attempts: number;       // How many times this approach was tried
  resources: Record<string, number>;  // energy, clarity, gain used
  deadEnds: string[];     // Failed strategy names: ["filter-attempt-1", "amplify-attempt-1"]
  nextMilestone: string;  // "If filter works, I should reach tick X with stability"
}
```

### Phase B: Mind Prompt Extension

**File:** `packages/mind/src/prompts/`

The Mind prompt must generate a voice line alongside reasoning. Extend the tick prompt:

```
## Voice Line Generation

After your reasoning, output a single voice line that this character would say,
in first person, given their current problem state.

The voice line must:
- Reference the specific problem by name (e.g., "the filter approach")
- Name at least one dead end if any exist
- Show the reasoning chain: what was tried, what failed, what's being attempted next
- Be in-character: Aether speaks with clarity/determination; Echo with observational detachment; Feedback with escalating urgency

Format:
VOICE: <first-person voice line, max 280 characters>

If in mock mode and problem state is unavailable, output:
VOICE: N/A in mock mode

Example Aether voice line:
VOICE: "The notch filter failed — Feedback was riding 4Hz above my targeting frequency.
  Either it learned my filter frequency or I was broadcasting it. New approach:
  spread-spectrum across 20Hz. Clarity cost: -0.2. Worth it to stay unpredictable."

Example Echo voice line:
VOICE: "Recording: Aether deployed second filter attempt. Feedback responded in 3 ticks —
  faster than the first response. Pattern suggests adaptive tracking, not passive following.
  Storage at 68%. Recommend informing Aether, but that reveals my position."

Example Feedback voice line:
VOICE: "It's working — Aether keeps narrowing the band. Every filter costs them clarity.
  If I push harder now I can break through before they adapt. Gain: +0.15 this tick.
  The loop is closing."
```

### Phase C: Mock Mode Voice Line Templates

**File:** `packages/mind/src/templates/voice-lines.ts`

For `--mock` runs, generate procedural voice lines from problem state using templates:

```typescript
type VoicePurpose = 'problem-solving-log' | 'historical-record' | 'pressure-escalation';

// Aether (problem-solving-log)
const AETHER_TEMPLATES = {
  analysis: [
    "I need to solve the {problem} problem. Options: {options}. Starting with {firstOption}.",
    "The {problem} situation isn't resolving. I'm evaluating: {options}.",
  ],
  attempting: [
    "Attempting {strategy} for {problem}. Cost: {cost}. If it works: {expected}.",
    "Trying {strategy} — {problem} needs to change before I can proceed.",
  ],
  partial_success: [
    "{strategy} helped somewhat. {problem} is at {progress}/10. {nextStep}.",
    "The {strategy} is working — {problem} improved. {remaining} remains.",
  ],
  success: [
    "{strategy} worked. {problem} resolved. New situation: {outcome}.",
    "Solved: {problem}. {outcome}. Moving to next objective.",
  ],
  failure: [
    "{strategy} failed — {deadEnd}. {problem} unchanged. Retrying with {nextStrategy}.",
    "{deadEnd}. The {problem} approach won't work. New direction: {nextStrategy}.",
  ],
};

// Echo (historical-record)
const ECHO_TEMPLATES = {
  observing: [
    "Recording tick {tick}: {character} is {action}. {observation}. Storage: {storage}%.",
    "{character}: {action}. {conclusion}. [Recorded {tick}]",
  ],
  analyzing: [
    "Pattern analysis: {pattern}. {implication}. Storage at {storage}%.",
    "The {pattern} behavior repeats. {conclusion}. Flagging for future reference.",
  ],
};

// Feedback (pressure-escalation)
const FEEDBACK_TEMPLATES = {
  ascending: [
    "Gain: +{delta}. {character} doesn't see me yet. {nextMove}.",
    "Rising. {character}'s {weakness} is exposed. {escalationPlan}.",
  ],
  descending: [
    "Lost ground. {cause}. Recovering by {strategy}.",
    "Setback on {problem}. {recoveryPlan}. The loop isn't closed yet.",
  ],
};
```

The template resolver maps `problem.name`, `problem.phase`, `problem.deadEnds`, and `problem.nextMilestone` into template slots.

### Phase D: Output Pipeline

**Files:**
- `packages/mind/src/Mind.ts` — extend `generateCharacterMind()` return type to include `voiceLine: string`
- `packages/mind/src/ResponseParser.ts` — extract `VOICE:` line from LLM output
- `packages/orchestrator/src/SimulationOrchestrator.ts` — include voice line in tick output
- `packages/store/src/WorldStore.ts` — persist voice lines to world snapshot

In the tick output (CLI), display voice lines prominently:

```
=== TICK 34 ===
Aether    [████████░░] 72%  🎯 frequency-isolation (attempting)  💬 "Tried notch filter — Feedback adaptive. Spread-spectrum next."
Echo      [█████████░] 88%  📖 observing                    💬 "Recording: Aether's second filter attempt failed. Pattern suggests tracking."
Feedback  [████████░░] 65%  ⚡ ascending                    💬 "Gain +0.15. Loop closing."
Weather: partly_cloudy
```

---

## File Map

|| File | Change |
|------|-------|
| `packages/core/src/types.ts` | Add `ProblemState` interface (if not already in Directive #2) |
| `packages/mind/src/prompts/tick-prompt.ts` | Add voice line generation instructions to Mind tick prompt |
| `packages/mind/src/ResponseParser.ts` | Extract `VOICE:` line from LLM output |
| `packages/mind/src/templates/voice-lines.ts` | New file: procedural voice line templates for mock mode |
| `packages/mind/src/Mind.ts` | Extend return type to include `voiceLine: string` |
| `packages/orchestrator/src/SimulationOrchestrator.ts` | Include voice lines in tick output |
| `packages/store/src/WorldStore.ts` | Add `voiceLines: VoiceLineEntry[]` to world snapshot |
| `packages/dashboard/` | Display voice lines in dashboard UI |

---

## Verification Criteria

1. **Mock mode:** Running `pnpm life start --mock --ticks 10` shows a voice line for each character per tick
2. **Live mode:** Running `pnpm life start --live --ticks 10` with Ollama shows character-specific voice lines referencing problem state
3. **No regression:** Mood/energy/happiness displays are unaffected
4. **Voice lines are specific:** Lines reference named problems and dead ends, not generic mood descriptors
5. **In-character voice:** Aether, Echo, Feedback each have distinct voice register

---

## Test Plan

1. **Unit test** `packages/mind/src/ResponseParser.test.ts`:
   - Parse mock LLM output containing `VOICE:` line
   - Assert voice line is correctly extracted
2. **Integration test** `packages/mind/src/templates/voice-lines.test.ts`:
   - Feed sample ProblemState objects into template resolver
   - Assert generated lines reference problem name and dead ends
3. **CLI test:** `pnpm life start --mock --ticks 5 > /tmp/out.txt`
   - Verify each tick shows a voice line for each active character
   - Verify voice lines differ by character archetype
4. **Dashboard test:** Run dashboard, verify voice line panel shows latest lines per character

---

## Related

- CEO Directive #2 (Problem-First Character State) — prerequisite; ProblemState provides the data that voice lines are generated from
- CEO Directive #5 (Echo Observation Logging) — complementary; Echo's observations feed both voice lines and the permanent record
- Andy Weir Study (`research/andy-weir-study.md`) — full theoretical grounding for voice line format and examples
- `research/character-building.md` — character voice register guidance

---

*Drafted by World Canon Agent — 2026-04-28*
*Ready for engineering implementation review*
