# Engineering Directive BL-2026-04-28: Narrator Duality — Echo vs Tempo

**Project:** beatfall-life (world simulation engine)
**Type:** World-Building Resolution
**Priority:** Low
**Status:** Draft
**Date:** 2026-04-28
**Applies to:** @beatfall-life/core, @beatfall-life/rules, simulation director

---

## Context

The `world/lore/character-roster.md` explicitly flags a contradiction:

> **Echo — The Outlier**
> Echo exists in the CLI fallback characters but NOT in the seed-expanded character set. Echo's canonical role as "The Main Narrator" conflicts with Tempo's "The Chronomancer."

Two characters claim the narrator role:
- **Echo** (FALLBACK_CHARACTERS, CLI-accessible): "Memory. Reflection. Reconstruction." — observes everything, records everything
- **Tempo** (seed-expanded, server-only): "I remember how the song began. I know how it ends." — time-aware, knows beginning and ending

This is a **first-class canonical conflict** that needs a resolution, not a deferral.

---

## Proposed Resolution

**Adopt a dual-narrator model** consistent with Beatfall's audio metaphor system:

| Concept | Echo | Tempo |
|---------|------|-------|
| **Audio analogy** | Reverb / delay | Time-stretch / tempo shift |
| **Narrative mode** | Present-tense observer | Past-future omniscient |
| **Scope** | Local, scene-by-scene | Global, arc-aware |
| **Voice** | Descriptive, sensory | Prophetic, arc-direction |
| **Canon status** | **CLI narrator** (accessible in all runs) | **Server narrator** (full simulation runs) |

**Both are canonical narrators.** Echo handles micro-narrative (what is happening now). Tempo handles macro-narrative (where the story is going). This mirrors how real audio productions use both a close microphone (Echo) and a director/temporal layer (Tempo).

### Implementation Tasks

1. **`@beatfall-life/rules`** — Add `narratorRole: 'echo' | 'tempo' | 'both'` to the CharacterDefinition schema. For CLI runs, default to `echo`. For full simulation runs with seed characters, enable `both`.

2. **`@beatfall-life/core`** — Add a `NarratorState` interface tracking `activeNarrators: NarratorId[]` and `currentNarrativeFocus: 'micro' | 'macro'`. Echo produces micro-narratives per tick. Tempo produces arc-level beat annotations.

3. **`simulation-director`** — When both narrators are active, emit Echo's tick-observation as `narrative.micro` and Tempo's arc-note as `narrative.macro`. They must never contradict; if conflict arises, Echo's micro wins (ground truth from direct observation).

4. **`lore/character-roster.md`** — Update the Echo/Tempo note to read: *"Echo and Tempo serve complementary narrator roles. Echo is the CLI-accessible present-tense observer. Tempo is the server-only arc-aware omniscient. Both are canonical."*

---

## Verification

- CLI simulation (`--chars aether echo feedback`) should produce Echo-narrated output
- Server simulation (seed characters, 22-char run) should produce dual-narrator output
- Echo's micro-narrative and Tempo's macro-narrative must never contradict on the same fact
- `character-roster.md` open question marked **RESOLVED**

---

## Open Question

**Should a third narrator exist?** The current seed roster has only Echo and Tempo. If a third narrator were added (e.g., "Requiem" as death-narrator, or "Harmony" as consensus-narrator), the dual model expands to a triad. Flag this as a future world-building decision — do not implement without canonical input.
