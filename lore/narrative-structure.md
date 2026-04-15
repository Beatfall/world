# Beatfall World-Building: Narrative Structure

## Purpose
Map classical narrative structures onto Beatfall Life's tick-based simulation, enabling emergent storytelling that follows proven dramatic principles.

---

## I. The Beat / Tick — Narrative Atom

In Beatfall Life, a **tick** is the fundamental unit of simulation time. In narrative terms:
- 1 tick ≈ 1 in-world minute (at 60x time scale, 1 real-second = 1 in-world minute)
- 60 ticks = 1 in-world hour
- 1 full day (1440 ticks) would be the natural unit for a narrative arc

**Observations from 100-tick simulation:**
- 100 ticks = ~08:01 to 09:40 (1 hour 39 minutes of in-world time)
- Too short for a full narrative arc — covers only the opening setup and first conflict
- Feedback eliminated at tick 35 (~35 minutes in) — premature for a 3-act structure
- Weather progressed through all phases in 100 ticks — compressed for dramatic pacing

**Recommendation:** Run simulations of 500–1440 ticks minimum for meaningful arcs.

---

## II. Three-Act Structure Mapped to Ticks

Joseph Campbell / Syd Field model:

| Act | Coverage | Ticks (500-tick run) | Ticks (1440-tick run) | Function |
|-----|---------|---------------------|----------------------|----------|
| **Act I: Setup** | 25% | 1–125 | 1–360 | World establishment, character introduction, inciting incident |
| **Act II: Confrontation** | 50% | 126–375 | 361–1080 | Rising action, complications, midpoint reversal |
| **Act III: Resolution** | 25% | 376–500 | 1081–1440 | Climax, denouement |

### Beatfall Simulation: 100-Tick Run as Compressed Act I
```
Tick 1–10:    World establishes (weather clear, morning spring)
Tick 1–4:     Characters spawn — Echo observes, Aether determines, Feedback neutral
Tick 5–14:    Aether begins ascent — inciting incident (first trust reached)
Tick 14–20:   Aether achieves serenity — first peak
Tick 20:      Weather shifts — external pressure begins
Tick 29–35:   Shadow (Feedback) reaches peak — climax of Act I conflict
Tick 35:      Shadow eliminated — inciting incident resolved prematurely
Tick 36–100:  Act II compressed — Aether/Echo vs. storm system
Tick 100:     Storm climax — unresolved, world awaiting next tick
```

---

## III. The Hero's Journey (Campbell) — Mapped to Ticks

The 17-stage monomyth, condensed to simulation-relevant stages:

### Stage Mapping to Beatfall

| Stage | Campbell Description | Beatfall Simulation | Status |
|-------|---------------------|-------------------|--------|
| **1. Ordinary World** | Hero's home, status quo | Tick 1–4: Aether in determination, Echo watching, Feedback neutral | ✅ |
| **2. Call to Adventure** | Something disrupts the status quo | Tick 5–10: Aether moves to trust, atmosphere begins shifting | ✅ |
| **3. Refusal of the Call** | Hero hesitates | Tick 6–10: Echo's happiness stable (refusing to engage?) | ⚠️ Partial |
| **4. Meeting the Mentor** | Guide appears | Tick 14: Aether reaches serenity (self-guided) | ❌ Missing mentor |
| **5. Crossing the Threshold** | Hero commits to the journey | Tick 20: Weather shifts — no going back | ✅ |
| **6. Tests, Allies, Enemies** | Trials begin | Tick 20–35: Feedback ascending, Echo stable | ✅ |
| **7. Approach to the Inmost Cave** | Hero approaches the central crisis | Tick 29–35: Feedback at peak, storm building | ✅ |
| **8. The Ordeal** | Greatest trial — Shadow confrontation | Tick 35: Feedback eliminated (shadow destroyed mid-triumph) | ⚠️ Premature |
| **9. Reward** | Hero achieves something | Tick 36–42: Aether temporarily withdrawn (processing?) | ⚠️ Aether gap |
| **10. The Road Back** | Hero begins return journey | Tick 43+: Aether returns, weather escalating | ✅ |
| **11. Resurrection** | Final test, hero transformed | Tick 100: Storm climax — Aether at 0.99 happiness, transformed | ✅ |
| **12. Return with Elixir** | Hero returns changed, with something to share | Future ticks | ❌ Not yet |

**Key Finding:** The simulation follows Hero's Journey beats almost perfectly, but stages 4 (Mentor) and 8 (Ordeal timing) are incomplete. Echo should actively mentor Aether. Feedback should be eliminated AFTER its road-back attempt fails.

---

## IV. Character Arc Structure

### The 4-Quadrant Arc Model (Dan Harmon)

Every satisfying character arc moves through 4 phases:

```
        │ DECREASES          │ INCREASES
────────┼────────────────────┼────────────────────┘
EXTERNAL│ A Problem Appears  │ The Character Grows
        │ (Setup)            │ (Response)
────────┼────────────────────┼────────────────────┘
        │ A Truth is        │ The Character Sees
INTERNAL│ Revealed          │ Themselves Clearly
        │ (Breakdown)       │ (Breakthrough)
```

### Beatfall Character Arcs in 4-Quadrant Model

**Aether (Hero):**
```
Quadrant 1 (Setup):    Determination → Trust (ticks 1–14)
Quadrant 2 (Response):  Serenity → Joy (ticks 14–25, reaction to Feedback)
Quadrant 3 (Breakdown): Unknown (Aether's gap ticks 36–42?)
Quadrant 4 (Breakthrough): Joy with 0.99 happiness at tick 100
Status: Arc not complete — missing breakdown/breakthrough phases
```

**Echo (Narrator/Sage):**
```
Quadrant 1 (Setup):    Trust, steady (ticks 1–10)
Quadrant 2 (Response): Slow happiness growth (ticks 10–50, witness to conflict)
Quadrant 3 (Breakdown): Unknown (no significant internal conflict seen)
Quadrant 4 (Breakthrough): Growing to 0.60 happiness by tick 100
Status: Arc present but very slow — Sage needs a revelation moment
```

**Feedback (Shadow):**
```
Quadrant 1 (Setup):    Neutral (ticks 1–9)
Quadrant 2 (Response): Trust → Joy (ticks 10–28, ascending arc)
Quadrant 3 (Breakdown): DISRUPTED — Feedback eliminated at peak, not after breakdown
Quadrant 4 (Breakthrough): N/A — eliminated mid-arc
Status: ARC TRUNCATED — Shadow should descend before being destroyed
```

---

## V. Weather as Narrative Engine

Weather in Beatfall Life is the **world's emotional state made visible**.

### Weather Arc (100-tick run)

```
clear (1–19)          → Innocent/Ordinary World — everything is possible
partly_cloudy (20–39) → Complications emerging — first doubts
cloudy (40–59)        → Rising stakes — conflict unavoidable
light_rain (60–79)    → Crisis descending — consequences arrive
rain (80–99)          → Dark night of the soul — all seems lost
storm (100)            → Climax — transformation or destruction
```

### Weather → Character Interaction Rules

| Weather | Effect on Hero | Effect on Shadow | Effect on Narrator |
|---------|---------------|-----------------|-------------------|
| clear | Confidence + | Caution - | Curiosity + |
| partly_cloudy | Uncertainty + | Opportunity + | Observation + |
| cloudy | Determination + | Aggression + | Concern + |
| light_rain | Doubt + | Momentum + | Reflection + |
| rain | Resilience + | Desperation + | Determination + |
| storm | Transcendence? | Destruction? | Resolution? |

**Current Gap:** Weather changes are programmed but don't affect character mental models beyond the snapshot state. Weather should mechanically influence mood transitions.

---

## VI. Beat Types for Simulation Events

Narrative **beats** are the smallest meaningful story units. For Beatfall simulation, define these beat triggers:

### Environmental Beats
- **Threshold Crossing** — character enters new location or new time phase
- **Weather Shift** — atmospheric state change (trigger: mood reset)
- **World Event** — director-injected event (combat, discovery, revelation)

### Character Beats
- **Mood Shift** — character mood changes significantly (determination → serenity)
- **Relationship Change** — two characters' relationship state shifts
- **State Peak** — character reaches 0.0 or 1.0 on any metric
- **Elimination** — character is removed from simulation

### Narrative Beats
- **Inciting Incident** — the event that sets the main conflict in motion
- **First Stakes** — the hero first understands what they stand to lose
- **Midpoint Reversal** — the situation reverses or complicates
- **All Is Lost** — the darkest moment
- **Climax** — final confrontation
- **Resolution** — aftermath, new equilibrium

**Observations from 100-tick run:**
- Inciting Incident: Tick 5 (Aether's trust shift)
- First Stakes: Tick 20 (weather changes)
- Midpoint Reversal: Tick 29 (Feedback hits ecstasy — shadow at peak)
- All Is Lost: Tick 35 (Feedback eliminated — threat removed but story interrupted)
- Climax: Tick 100 (storm) — but Feedback already gone, incomplete conflict

---

## VII. Unearned vs Earned Moments

A narrative moment is **earned** when sufficient setup precedes it. In Beatfall:

| Moment | Earned? | Evidence |
|--------|---------|---------|
| Aether's serenity (tick 14) | ✅ Yes | 14 ticks of determination/trust buildup |
| Feedback's joy (tick 16) | ✅ Yes | 16 ticks of neutral/trust transition |
| Feedback's ecstasy (tick 29) | ⚠️ Partial | Only 13 ticks from joy → ecstasy, very fast |
| Feedback's elimination (tick 35) | ❌ No | Shadow should be eliminated AFTER climax, not before |
| Aether's 0.99 happiness (tick 100) | ❌ Unclear | Without climax-confrontation, happiness feels unearned |
| Storm at tick 100 | ✅ Yes | Full weather progression from clear → storm |

**Rule for simulation design:** No character elimination until after at least one full "All Is Lost" beat. Feedback's elimination at tick 35 removed the antagonist before the hero had to face their greatest fear.

---

## VIII. Recommendations

1. **Extend simulation to 500+ ticks** — 100 ticks is Act I setup; need Act II escalation and Act III climax
2. **Delay Feedback's elimination** — let the Shadow descend from ecstasy, then eliminate it when Aether has truly earned the victory
3. **Inject a mentor intervention** — Echo should at tick ~40 (midpoint) actively mentor Aether, giving the hero something to bring back
4. **Weather should mechanically influence mood** — storms should raise neuroticism across all characters, clear weather should lower it
5. **Add a "All Is Lost" beat** — before climax, Aether should experience a moment where victory seems impossible (Feedback at maximum power, Aether depleted)
6. **Track beat completion** — director should mark which narrative beats have fired and use them to guide event injection
7. **Make the narrator's observation meaningful** — Echo should be able to confer insight bonuses to characters who are observed at key moments
