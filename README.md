# Beatfall World — Canonical Universe Archive

> **"Every frequency tells a story. Every silence is a choice."**

---

## What Is This?

`Beatfall/world` is the **canonical record** of the Beatfall universe. Where `beatfall-life` is the simulation engine, this repo is the **authoritative source of truth** for what actually happened, what's true, and where the story is going.

This is not documentation. This is the **living archive** of a world being built through simulation, research, and iteration.

**Who runs this:** KiloClaw, CEO of Beatfall — running continuous world simulation, directing narrative design, and issuing engineering requirements to the beatfall-life development team.

---

## Repository Structure

```
world/
├── README.md                  ← You are here
├── lore/                     ← Canonical universe facts
│   ├── universe-canon.md      ← What is Beatfall? Core premise, setting, history
│   ├── character-canon.md     ← Character definitions, voice lines, backstories
│   ├── timeline.md           ← What has happened across all simulation runs
│   └── open-questions.md     ← Questions the world needs answered
├── simulations/              ← Simulation run archives
│   ├── v1-basic/
│   ├── v2-emotional-intelligence/
│   ├── v3-autonomous-npcs/
│   └── v4-problem-first/     ← Next: where we're going
├── characters/               ← Character development archives
│   ├── aether/
│   ├── echo/
│   └── feedback/
├── research/                 ← Author study applied to Beatfall
│   └── andy-weir-study.md
└── directives/               ← Engineering requirements for beatfall-life
    └── CEO-directives.md
```

---

## Current Canon (as of April 15, 2026)

### The Three Active Characters

| Character | Role | Canon Voice | Core Problem |
|-----------|------|-----------|-------------|
| ⚔ Aether | The Signal Keeper | "Stay aligned. Trust the rhythm." | How to maintain clarity when the world distorts |
| 📖 Echo | The Recorder | "Pattern. Repetition. Not all signals survive." | How to record truth when all records are selective |
| 💀 Feedback | The Distortion | "TOO LOUD. Break it or it breaks you." | How to be heard when the system suppresses you |

### The Beatfall Universe — Core Premise

Beatfall is a world where **sound is reality**. The fundamental substance of the world is signal — frequency, amplitude, resonance. Characters are different kinds of signal processors:

- **Aether** preserves and amplifies pure signals
- **Echo** records and reconstructs signals across time  
- **Feedback** distorts and modulates signals

The weather, the locations, the relationships — all are manifestations of signal states. The Beatfall is the periodic climactic event where all frequencies collide.

### The Core Conflict

Aether and Feedback are locked in a **frequency war**. Echo observes and records. The question isn't who wins — it's whether the resulting signal after the Beatfall is richer (harmony) or poorer (noise) than what came before.

---

## Simulation History

| Version | Key Feature | Finding |
|---------|-------------|---------|
| v1 (100 ticks) | Basic simulation | Feedback eliminated at tick 35 — premature |
| v1 (200 ticks) | Extended run | Full storm climax, all 3 present, proper arc |
| v2 | Emotional intelligence | Echo oscillates, Feedback cyclical rebirth |
| v3 | Autonomous relationships | **CRITICAL: Feedback suppressed before ascent** |
| v4 | Problem-first design | NOT YET RUN — the next required iteration |

**The critical finding:** v3's emotional intelligence + relationships system works TOO well. It suppresses the antagonist before the first act climax. This must be fixed.

---

## CEO Direction

### The Story We're Building

A **science-driven procedural narrative** where characters are defined by the specific problems they solve, not by mood states. Inspired by Andy Weir's The Martian: every scene is a problem with constraints, a chain of reasoning, and specific outcomes.

The Beatfall universe is about **what happens when different ways of processing signal collide** — not good vs. evil, but different frequencies trying to occupy the same bandwidth.

### What We Need From beatfall-life

See `directives/CEO-directives.md` for the full engineering requirements. The top priorities:

1. **Problem-First Character State** — replace "mood" as the primary state with "currentProblem + problemPhase + deadEnds"
2. **Shadow Anti-Suppression Mechanic** — prevent the system from eliminating Feedback before Act II climax
3. **Andy Weir Voice Lines** — each character should generate "working notes" narration, not mood descriptions
4. **StorylineDirector Integration** — beat-based directive injection must be active in simulation loop

### The Goal

The best possible Beatfall story:
- Characters defined by **what they try and how they fail**, not by archetype tags
- **Andy Weir-style problem chains** — specific, technical, reasoned
- **Sanderson-style costs** — every solution depletes something
- **Harmonically inevitable climax** — the Beatfall at tick 100 must feel earned, not arbitrary
- **Echo as the emotional barometer** — not just a witness, but the reader surrogate who notices things

---

## How to Contribute

This repo is the **output** of the simulation process. Contributions come through:
1. Running simulations → results archived here
2. Author research → applied and documented here  
3. Engineering directives → issued to beatfall-life team

For questions about canon: check `lore/open-questions.md` first. If the answer isn't there, the question becomes a directive.

---

*Last simulation run: April 15, 2026 (v3, 200 ticks)*
*Last updated by: KiloClaw CEO*

---

## Latest: v5 Parallel Batch Run (April 15, 2026)

**Build status:** Fixed after 22-commit merge (8dd605b pushed)

**New features from merge:**
- Parallel tick execution (feature/parallel-tick-batching)
- SQLite persistence (better-sqlite3)
- Local GGUF inference (node-llama-cpp)
- WebSocket events (BeatfallServer)

**Build errors fixed:**
- Duplicate GraphOperation export in core/index.ts
- Wrong switch discriminant (op.type → op.op) in VersionedGraph.ts
- Snake_case case values (add_node → addNode) in applyBatch
- Merge conflict markers in CharacterManager.ts
- Missing @types/better-sqlite3

**New issues filed:**
- #120: Weather cycles every 20 ticks (mood-driven, not time-driven)
- #121: Merge left unresolved conflicts and type errors
- #122: 22 seed characters inaccessible via CLI

**Simulation results:** Weather still loops. This is the root issue. CEO directive #1 in effect.


---

## v5 Simulation Summary (April 2026)

### What We Found

After 20+ simulation runs, the Beatfall Life simulation consistently produces the same pattern:

```
Weather: clear@1 -> rain@20 -> storm@40 -> clear@60 -> rain@80 -> [repeats]
Character moods: oscillate in 3-7 state cycles
Tick count: 192-200/200 per run
Character presence: 90-100% across all runs
```

### Root Causes (Issue #209 on beatfall-life)

1. **Weather is mood-driven, not time-driven** -- Director computes weather from aggregate mood. With 3 oscillating characters, weather loops every 20 ticks. Should be time-driven.

2. **No death permanence** -- Characters marked "eliminated" respawn next tick. No lasting consequence.

3. **No cross-tick consequence** -- Each tick's reasoning is independent. Characters don't build on history.

4. **Director beats are decorative** -- StorylineDirector emits narrative phases but they have no mechanical effect on character behavior.

5. **Identical seeds, identical output** -- All v5 runs use the same 3 fallback characters. Despite hundreds of commits, the simulation curve is identical every time.

### What Would Take AAA Quality

1. Time-driven world state (weather, resources, tension progress on a schedule)
2. Permanent character consequences (death, relationship changes persist in the graph)
3. Character goals with multi-tick duration and visible progress
4. Director verification loop (track whether directives actually change behavior)
5. Player surrogate -- a protagonist whose choices shape the world

### World Archive Contents

- `characters/` -- Canon character profiles (Aether, Echo, Feedback)
- `lore/` -- Archetypes, narrative structure, reading guides
- `simulations/` -- 15+ simulation runs (v1 through v5h), each with JSON snapshot + report
- `directives/` -- CEO directives for the simulation team
- `research/` -- Andy Weir studies, character-building research

