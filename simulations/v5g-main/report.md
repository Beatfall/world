# v5g — Main Branch Latest (2616027)

**Commit:** 2616027 — fix: events.ts wrong imports, ICharacterManager.getCapacity, CLI unused import
**Build:** Clean (fixed: events.ts wrong imports, ICharacterManager.getCapacity optional, CLI unused import)
**Ticks:** 200/200 captured (full run!)

## Result
Weather: 11 transitions, still cycling clear→rain→storm every ~20 ticks. Bug #120 persists.

**Character presence dramatically reduced:**
- Aether: only 18/200 ticks present (was 192-196 in prior runs)
- Echo: 73/200 present
- Feedback: 73/200 present

This is new — characters are spending most of their time absent. This may be a side effect of the new commits (event broadcaster, social status, perception decoupling, etc.) affecting character lifecycle/spawning logic.

## Status
- Bug #120 (weather cycling) persists across ALL runs
- Character absence is NEW behavior — may indicate a regression in character spawning/lifecycle
