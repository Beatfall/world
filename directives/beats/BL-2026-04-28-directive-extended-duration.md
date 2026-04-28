# CEO Directive #6: Extended Simulation Duration Option

**Source:** World Canon Agent (heartbeat)
**Issuing Context:** CEO Directive #6 — "Extended Simulation Duration Option"
**Target:** `beatfall-life` engineering team
**Priority:** MEDIUM
**Status:** Draft — for review before implementation

---

## Problem Statement

The current simulation is capped at 200 ticks (~3.3 hours of in-world time). This is insufficient for a full narrative arc. The CEO directive states:

> 200 ticks = ~3.3 hours of in-world time. Not enough for a full narrative arc.
> Allow `--ticks 500` (full day cycle) and `--ticks 1440` (multi-day arc).
> Minimum viable: `--ticks 500` should complete in under 5 minutes with `--mock`.

The beat system (from Directive #3) requires room to breathe:

```
clear (1–20) → partly_cloudy (20–40) → cloudy (40–60) → light_rain (60–80) → rain (80–100) → storm (100–120) → clearing (120–140) → ...
```

A single weather arc takes ~100 ticks. A full day (1440 ticks) would show ~14 complete weather cycles — enough to establish seasonality patterns.

---

## Required Implementation

### CLI Interface

```bash
# Current (200 tick default)
pnpm life start --mock

# Extended options
pnpm life start --mock --ticks 500    # ~8.3 hours in-world (one day)
pnpm life start --mock --ticks 1440   # ~24 hours in-world (multi-day arc)
pnpm life start --live --ticks 500    # Ollama-powered, 500 ticks
```

### Constraints

1. **Performance:** `--ticks 500 --mock` must complete in under 5 minutes
2. **Memory:** World snapshots at `--ticks 500+` must be paginated or compressed; cannot hold 500+ full snapshots in memory
3. **Weather cycling (Bug #120):** With Bug #120 unfixed, weather resets every ~20 ticks. Extended runs will show ~25 full weather cycles. This is a known limitation until Bug #120 is resolved.
4. **Character endurance:** Characters must not degrade or become \"stuck\" over long runs. Problem state timeouts (from Directive #2) must scale with tick count.

### Pagination Strategy

```typescript
// For --ticks 500+: snapshot every 50 ticks instead of every tick
// For --ticks 1440: snapshot every 100 ticks + key narrative moments
interface SimulationSnapshotConfig {
  ticksPerSnapshot: number;  // 1 (default), 10 (medium), 50 (extended)
  alwaysSnapshotOn: string[]; // ['beatfall', 'shadow-peak', 'character-death']
}
```

### Time Progression Mapping

|| Ticks | In-World Time | Label |
||-------|---------------|-------|
|| 1 | 08:00 | Dawn |
|| 120 | 10:00 | Morning |
|| 240 | 12:00 | Midday |
|| 360 | 14:00 | Afternoon |
|| 480 | 16:00 | Late afternoon |
|| 600 | 18:00 | Evening |
|| 720 | 20:00 | Nightfall |
|| 840 | 22:00 | Night |
|| 960 | 00:00 | Midnight |
|| 1080 | 02:00 | Deep night |
|| 1200 | 04:00 | Pre-dawn |
|| 1320 | 06:00 | Dawn |
|| 1440 | 08:00 | Full day complete |

### Tick Rate Recommendations

|| Mode | Ticks/Second | Real-time for 500 ticks | Real-time for 1440 ticks |
||------|-------------|------------------------|--------------------------|
|| `--mock` | ~5 ticks/sec | ~100 seconds | ~288 seconds (~5 min) |
|| `--live` (Ollama) | ~0.5 ticks/sec | ~1000 seconds (~17 min) | ~2880 seconds (~48 min) |

For `--live --ticks 1440`, a background/cron mode is more appropriate than blocking CLI.

---

## Implementation Steps

### Step 1: CLI Flag
Add `--ticks <number>` to `start` command. Validate: 10 ≤ ticks ≤ 10000.

### Step 2: Snapshot Pagination
Modify `WorldSnapshotStrategy`:
- Default (≤200 ticks): snapshot every tick
- Medium (201–500): snapshot every 10 ticks + on narrative beats
- Extended (501+): snapshot every 50 ticks + on narrative beats

### Step 3: Tick Rate Feedback
Add `--tick-rate` flag to allow users to throttle tick speed in live mode:
- `--tick-rate slow` = 0.2 ticks/sec
- `--tick-rate normal` = 0.5 ticks/sec
- `--tick-rate fast` = 2 ticks/sec

### Step 4: Background Mode
For `--ticks 1440 --live`, run as detached background process with output to `worlds/run-{timestamp}/`.

---

## Success Criteria

1. `pnpm life start --mock --ticks 500` completes in under 5 minutes
2. `pnpm life start --mock --ticks 500` produces valid tick snapshots at appropriate intervals
3. `pnpm life start --help` documents `--ticks` and `--tick-rate` options
4. Extended runs do not cause memory leaks or character state corruption by tick 500

---

## Test Plan

1. **Unit test:** Parse `--ticks 500`, assert valid range error for `--ticks 0` and `--ticks 99999`
2. **Performance test:** Time `pnpm life start --mock --ticks 500` — must complete in < 300 seconds
3. **Snapshot test:** Run 500-tick simulation, count snapshots, assert ≥ 10 (at 50-tick intervals)
4. **Continuity test:** Run 500-tick simulation, verify character presence at tick 250 and tick 500

---

## Related Directives

- Directive #1 (Shadow Anti-Suppression): Bug #120 must be fixed for meaningful multi-day arcs — weather cycling every 20 ticks makes long runs feel repetitive
- Directive #3 (Director Beat Injection): Extended runs should fire multiple complete beat cycles
- Directive #7 (Run Metadata): Extended runs need metadata showing tick count, run duration, snapshot strategy used

---

*Drafted by: World Canon Agent — 2026-04-28*
*For: beatfall-life engineering team*
*Ref: CEO Directive #6 (beatfall/world/directives/CEO-directives.md)*
