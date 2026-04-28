# Beatfall — Canonical Beat Registry

> **Purpose:** Define all narrative beat types, their trigger conditions, and their firing status across simulation runs. This is the reference document for Directive #8 (Beat Completion Report) and Directive #3 (Director Beat Injection).
>
> **Source of truth:** Simulation observations take precedence over design intent.

---

## I. Beat Type Definitions

Each beat type has:
- **ID** — canonical slug
- **Name** — human-readable label
- **Narrative Stage** — Hero's Journey / Three-Act mapping
- **Trigger Condition** — when the Director should fire this beat
- **Tick Window** — expected simulation tick range (for 200-tick run)
- **Effect** — what firing the beat changes in the simulation

---

### Environmental Beats

#### `world-init`
| Field | Value |
|-------|-------|
| **Name** | World Initialized |
| **Narrative Stage** | Act I — Setup |
| **Trigger Condition** | Simulation tick = 1 |
| **Tick Window** | tick 1 |
| **Effect** | 5 locations spawned, 3 characters present, weather = clear |

---

#### `weather-shift`
| Field | Value |
|-------|-------|
| **Name** | Weather Transition |
| **Narrative Stage** | Ongoing |
| **Trigger Condition** | Aggregate character mood crosses threshold |
| **Tick Window** | tick ~20, ~40, ~60, ~80, ~100 |
| **Effect** | Weather state advances (clear → partly_cloudy → cloudy → light_rain → rain → storm) |

---

### Character Beats

#### `mood-shift`
| Field | Value |
|-------|-------|
| **Name** | Character Mood Transition |
| **Narrative Stage** | Ongoing |
| **Trigger Condition** | Any character's mood.valence changes by > 0.3 |
| **Tick Window** | Any tick |
| **Effect** | MentalModelEngine updates character mood state |

---

#### `character-spawn`
| Field | Value |
|-------|-------|
| **Name** | Character Spawned |
| **Narrative Stage** | Act I — Setup |
| **Trigger Condition** | New character enters simulation |
| **Tick Window** | tick 1 (initial), tick > 1 (respawn) |
| **Effect** | Character added to active set, relationships initialized |

---

#### `character-eliminated`
| Field | Value |
|-------|-------|
| **Name** | Character Eliminated |
| **Narrative Stage** | Varies |
| **Trigger Condition** | Character safety drops to 0 (subject to Shadow Anti-Suppression gate) |
| **Tick Window** | Any tick |
| **Effect** | Character removed from active set; isIncapacitated = true |

---

### Narrative Structure Beats

#### `inciting-incident`
| Field | Value |
|-------|-------|
| **Name** | Inciting Incident |
| **Narrative Stage** | Hero's Journey #2 — Call to Adventure |
| **Trigger Condition** | Aether first achieves trust (mood valence > 0.5) OR first weather shift fires |
| **Tick Window** | tick 5–20 |
| **Effect** | World is no longer in equilibrium — hero has committed to action |

**Observed firings:**
| Run | Fired? | Tick |
|-----|--------|------|
| v1-basic | ✅ Yes | ~5 |
| v2-emotional-intelligence | ✅ Yes | ~10 |
| v3-autonomous-npcs | ✅ Yes | ~8 |
| v5d-latest | ✅ Yes | ~5 |
| v5e-pr149 | ✅ Yes | ~5 |

---

#### `first-stakes`
| Field | Value |
|-------|-------|
| **Name** | First Stakes Revealed |
| **Narrative Stage** | Hero's Journey #6 — Tests, Allies, Enemies |
| **Trigger Condition** | Weather shifts to partly_cloudy OR Feedback reaches joy |
| **Tick Window** | tick 15–25 |
| **Effect** | External pressure begins — characters can no longer ignore the conflict |

**Observed firings:**
| Run | Fired? | Tick |
|-----|--------|------|
| v1-basic | ✅ Yes | ~20 (weather shift) |
| v2-emotional-intelligence | ✅ Yes | ~16 (Feedback joy) |
| v3-autonomous-npcs | ✅ Yes | ~20 (weather shift) |
| v5d-latest | ✅ Yes | ~20 (weather shift) |
| v5e-pr149 | ✅ Yes | ~20 (weather shift) |

---

#### `shadow-rise`
| Field | Value |
|-------|-------|
| **Name** | Shadow Ascendant |
| **Narrative Stage** | Hero's Journey #7 — Approach to the Inmost Cave |
| **Trigger Condition** | Feedback mood valence > 0.7 OR Feedback gain floor increases |
| **Tick Window** | tick 25–40 |
| **Effect** | Shadow (Feedback) gains power floor — harder to suppress |

**Observed firings:**
| Run | Fired? | Tick |
|-----|--------|------|
| v1-basic | ⚠️ Partial | ~16 (Feedback joy, but elim @35 before full ascendancy) |
| v2-emotional-intelligence | ✅ Yes | ~60 (sustained conflict) |
| v3-autonomous-npcs | ⚠️ Partial | ~29 (Feedback ecstasy) |
| v5d-latest | ❌ No | Feedback elim @35 before rise |
| v5e-pr149 | ❌ No | Feedback elim @35 before rise |

---

#### `shadow-peak`
| Field | Value |
|-------|-------|
| **Name** | Shadow at Peak |
| **Narrative Stage** | Hero's Journey #8 — The Ordeal |
| **Trigger Condition** | Feedback mood valence > 0.8 AND arousal > 0.8 — Director must detect and emit |
| **Tick Window** | tick 65–85 |
| **Effect** | **Unblocks Shadow elimination** (Directive #1 gate); Director emits narrative_beat event |

**Observed firings:**
| Run | Fired? | Tick |
|-----|--------|------|
| v1-basic | ❌ No | Eliminated at peak @tick 29, never began descent |
| v2-emotional-intelligence | ✅ Yes | @tick ~100 (200-tick run, climax with Shadow present) |
| v3-autonomous-npcs | ❌ No | Eliminated @tick 97, peak reached but no descent cycle |
| v5d-latest | ❌ No | Eliminated @tick 35, peak not reached |
| v5e-pr149 | ❌ No | Eliminated @tick 35, peak not reached |

**CEO Directive #1 (Shadow Anti-Suppression) requires this beat to fire before any Shadow elimination is permitted.**

---

#### `all-is-lost`
| Field | Value |
|-------|-------|
| **Name** | All Is Lost |
| **Narrative Stage** | Hero's Journey #9 — Reward / The Road Back |
| **Trigger Condition** | Aether safety < 0.2 AND Feedback is present AND shadow-peak has fired |
| **Tick Window** | tick 85–95 |
| **Effect** | Hero at minimum; Shadow at maximum — climax is imminent |

**Observed firings:**
| Run | Fired? | Tick |
|-----|--------|------|
| v1-basic | ❌ No | Shadow gone before all-is-lost moment |
| v2-emotional-intelligence | ⚠️ Unclear | Tick ~90 had Aether safety low but Shadow still present |
| v3-autonomous-npcs | ❌ No | Shadow elim @97, all-is-lost requires Shadow at peak |
| v5d-latest | ❌ No | No valid climax condition |
| v5e-pr149 | ❌ No | No valid climax condition |

---

#### `beatfall`
| Field | Value |
|-------|-------|
| **Name** | Beatfall — The Storm Climax |
| **Narrative Stage** | Hero's Journey #11 — Resurrection |
| **Trigger Condition** | tick 100 OR Director emits climax event |
| **Tick Window** | tick 100 |
| **Effect** | Weather = storm; highest emotional intensity; all character arcs converge |

**Observed firings:**
| Run | Fired? | Tick |
|-----|--------|------|
| v1-basic | ✅ Yes | tick 100 (weather storm, but Shadow absent) |
| v2-emotional-intelligence | ✅ Yes | tick 100 (weather storm, Shadow present — proper climax) |
| v3-autonomous-npcs | ✅ Yes | tick 100 (weather storm, but Shadow elim @97) |
| v5d-latest | ✅ Yes | tick 100 (weather storm, but Shadow absent) |
| v5e-pr149 | ✅ Yes | tick 100 (weather storm, but Shadow absent) |

**Note:** beatfall fires in ALL runs. The issue is that in most runs, the Shadow is absent at tick 100, making the climax hollow.

---

#### `resolution`
| Field | Value |
|-------|-------|
| **Name** | Resolution |
| **Narrative Stage** | Hero's Journey #12 — Return with Elixir |
| **Trigger Condition** | All prior beats have fired AND simulation tick > 100 |
| **Tick Window** | tick 101–200 |
| **Effect** | World stabilizes; new equilibrium established |

**Observed firings:**
| Run | Fired? | Tick |
|-----|--------|------|
| v1-basic | ❌ No | Run ended @tick 100 |
| v2-emotional-intelligence | ❌ No | Run ended @tick 200 |
| v3-autonomous-npcs | ❌ No | Run ended @tick 200 |
| v5d-latest | ❌ No | Run ended @tick 100 |
| v5e-pr149 | ❌ No | Run ended @tick 100 |

---

## II. Beat Completion Matrix

| Beat | v1 | v2 | v3 | v5d | v5e | Required for Resolution? |
|------|----|----|----|-----|-----|--------------------------|
| `world-init` | ✅1 | ✅1 | ✅1 | ✅1 | ✅1 | No |
| `inciting-incident` | ✅5 | ✅10 | ✅8 | ✅5 | ✅5 | No |
| `first-stakes` | ✅20 | ✅16 | ✅20 | ✅20 | ✅20 | No |
| `shadow-rise` | ⚠️16 | ✅60 | ⚠️29 | ❌35 | ❌35 | No |
| `shadow-peak` | ❌35 | ✅~100 | ❌97 | ❌35 | ❌35 | **Yes** |
| `all-is-lost` | ❌ | ⚠️~90 | ❌97 | ❌ | ❌ | **Yes** |
| `beatfall` | ✅100 | ✅100 | ✅100 | ✅100 | ✅100 | **Yes** |
| `resolution` | ❌ | ❌ | ❌ | ❌ | ❌ | — |

**Legend:** ✅ = fired, ❌ = did not fire, ⚠️ = partial (reached condition but context wrong)

**Key finding:** v2 is the only run with a complete beat arc (shadow-peak + beatfall both fired with Shadow present). All other runs have hollow climaxes because the Shadow is eliminated before shadow-peak can fire.

---

## III. Directive Coverage Map

| Directive | Beat Type | Status |
|-----------|-----------|--------|
| Directive #1 — Shadow Anti-Suppression | `shadow-peak` gate | Draft complete |
| Directive #2 — Problem-First State | Mood-as-consequence (upstream of all beats) | Draft complete |
| Directive #3 — Director Beat Injection | All beat types | Draft complete |
| Directive #4 — Voice Line Generation | Beat-triggered character voice | Draft complete |
| Directive #5 — Echo Observation Logging | Beat-triggered narrator insight | Draft complete |
| Directive #6 — Extended Duration | Enables `resolution` beat (requires 200+ ticks) | Draft complete |
| Directive #7 — Run Metadata | Metadata per run | Draft complete |
| Directive #8 — Beat Completion Report | All beat types | Draft complete |

---

## IV. Beat Verification Criteria (v0.2.0)

A v0.2.0 simulation run is considered **narratively complete** when:

1. ✅ `shadow-peak` fires before tick 90 (Feedback at valence > 0.8, arousal > 0.8)
2. ✅ Feedback is present at tick 100 (storm climax)
3. ✅ `all-is-lost` fires (Aether safety < 0.2, Shadow present)
4. ✅ `beatfall` fires at tick 100
5. ✅ `resolution` fires between tick 101–200
6. ✅ Beat Completion Report (Directive #8) is generated post-run

---

## V. Open Questions

1. **Beat priority ordering:** If two beats fire in the same tick, which takes precedence? (e.g., shadow-peak and all-is-lost both possible at tick ~90)
2. **Beat deduplication:** Can the same beat fire twice in one run? (e.g., two separate `mood-shift` events — yes, but structural beats like `shadow-peak` should only fire once)
3. **Beat skipping:** If a beat's trigger condition is met but the Director misses it (no emit), should the beat be retroactively marked as fired based on tick log analysis?
4. **Beat categories:** Should structural narrative beats (shadow-peak, all-is-lost, beatfall, resolution) be distinguished from ongoing mechanical beats (mood-shift, weather-shift)?

---

*Maintained by: World Canon Agent*
*Source: CEO-directives.md, timeline.md Section VIII, narrative-structure.md*
*Last updated: 2026-04-30T05:15:00Z*
