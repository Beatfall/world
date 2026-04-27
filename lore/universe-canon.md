# Beatfall Universe — Canonical Reference

## Purpose
Establish the canonical facts of the Beatfall universe as derived from character definitions, simulation observations, and project documentation.

---

## I. Core Premise

**Beatfall** is a universe built around music, rhythm, and the interplay of signal and noise. The name evokes both "beat" (rhythm, music) and "fall" (descent, loss, the drop in a track). Characters exist as living embodiments of musical/sonic concepts.

**The fundamental tension of Beatfall:** Clarity vs. Distortion. The universe is a world of audio signals — heroes preserve and amplify pure signals, antagonists corrupt and distort them.

---

## II. The Beatfall World — Environmental Canon

From simulation observations:

### Time System
- **Tick:** Base simulation unit, 1 tick = 1 in-world minute (at 60x time scale)
- **Day:** 1440 ticks (24 hours)
- **Phase:** morning, afternoon, evening, night (each ~6 hours = 360 ticks)
- **Season:** spring, summer, autumn/fall, winter (not yet observed in simulation)
- **Year:** Not yet established in simulation

### Weather System
Weather follows emotional arc rules (see narrative-structure.md). Observed states:
- `clear` — default, stable
- `partly_cloudy` — first complication
- `cloudy` — rising tension
- `light_rain` — crisis in progress
- `rain` — deep crisis
- `storm` — climax

**Hypothesis:** Weather correlates with aggregate emotional state of active characters. Storm = collective emotional climax.

### Locations

The Beatfall world contains 5 canonical locations, each mapped to a frequency band in the signal spectrum. Characters move between locations by "shifting frequency."

|| ID | Name | Frequency Band | Atmosphere | Notes ||
||----|------|---------------|------------|-------|
|| loc_0 | **The Source** | 20Hz–20kHz (full spectrum) | sacred, origin point | Where characters spawn. The pristine signal origin. |
|| loc_1 | **The Midfield** | 200Hz–8kHz (vocal range) | inhabited, active | Echo is most effective here. Aether's voice carries clearly. |
|| loc_2 | **The Low Shelf** | 20Hz–250Hz (bass range) | dark, resonant, dangerous | Feedback's native frequency. Dangerous for heroes to enter without protection. |
|| loc_3 | **The High Pass** | 8kHz–20kHz (air and shimmer) | thin, isolating | Where Clarity lives. Easy to get lost. High ground for Aether's filtering. |
|| loc_4 | **The Notch** | 2kHz–4kHz (presence range) | contested, crossover | The battlefield. All three characters' signals overlap here. Site of the central conflict. |

**Location Movement:** Characters shift frequency to change location. Moving to The Low Shelf (Feedback's domain) without gain reduction is dangerous. Moving to The High Pass gives Aether tactical advantage. Echo functions across all locations but is most present in The Midfield.

**Recommendation:** Add location exploration to simulation — characters can move between locations, triggering events.

---

## III. Characters — Canonical Definitions

### ⚔ Aether — The Celestial DJ
```yaml
role: hero
archetype: The Celestial DJ
personality:
  openness: 0.8        # High — receptive to new signals
  conscientiousness: 0.7 # Organized, purposeful
  extraversion: 0.4   # Introverted — acts from inner conviction
  agreeableness: 0.8  # Collaborative, harmonious
  neuroticism: 0.2    # Emotionally stable
backstory: "Clarity. Direction. Intent. Speaks to heroes directly."
voice: "Stay aligned. Don't follow the noise. You already know the rhythm. Trust it."
```

**Interpretation:** Aether is a spiritual/musical guide. "Stay aligned" = stay in tune. "Don't follow the noise" = avoid distortion. "You already know the rhythm" = trust your inner pitch/meter. Aether speaks to other heroes, not to the world.

**Simulation Arc (100 ticks):** determination → serenity → ecstasy → joy. High energy but emotionally stable. Hero archetype.

---

### 📖 Echo — The Main Narrator
```yaml
role: narrator
archetype: The Main Narrator
personality:
  openness: 0.9        # Maximum — observes everything
  conscientiousness: 0.8 # Systematically records
  extraversion: 0.2    # Very introverted — watches rather than participates
  agreeableness: 0.6   # Neutral — observes all sides
  neuroticism: 0.4     # Slightly neurotic — the weight of knowing
backstory: "Memory. Reflection. Reconstruction."
voice: "It started as a pattern. Small. Repeating. Not all signals survive intact."
```

**Interpretation:** Echo is the chronicler of Beatfall. "Memory" = it's a recording. "Reflection" = it's the world reflecting on itself. "Reconstruction" = it builds narrative from raw data. "Not all signals survive intact" = information is lost in transmission, history is incomplete.

**Simulation Arc (100 ticks):** Steady `trust` throughout, slow happiness growth. The witness who outlasts conflict.

---

### 💀 Feedback — The Bass Hacker
```yaml
role: antagonist
archetype: The Bass Hacker
personality:
  openness: 0.7        # Creative in a destructive way
  conscientiousness: 0.3 # No discipline
  extraversion: 0.8   # Loud, aggressive presence
  agreeableness: 0.2  # Antagonistic
  neuroticism: 0.7     # Volatile emotions
backstory: "Distortion. Pressure. Escalation. Real-time combat reaction."
voice: "Too loud—TOO LOUD— Gain rising. No control. Break it or it breaks you."
```

**Interpretation:** Feedback is audio feedback loop made literal — the screeching howl that happens when a mic picks up its own output. "Distortion" = corruption. "Pressure" = intensity. "Escalation" = runaway loop. "Real-time combat reaction" = it responds instantly and aggressively.

**Simulation Arc (100 ticks):** Started neutral, oscillated between neutral/trust, hit joy at tick 16, ecstasy at tick 29, eliminated at tick 35 mid-ecstasy. The antagonist whose threat was cut short.

---

## IV. Character Relationships (Inferred from Simulation)

### Aether ↔ Echo
- **Relationship:** Echo observes Aether's journey. Aether benefits from being observed (insight bonus?).
- **Inferred dynamic:** Echo is Aether's external memory. Aether acts, Echo records.
- **Missing:** No active mentor/mentee dynamic. Echo should guide, not just observe.

### Aether ↔ Feedback
- **Relationship:** Protagonist/Antagonist. Fundamental opposition.
- **Inferred dynamic:** Aether's clarity vs. Feedback's distortion. The central conflict.
- **Outcome (tick 35):** Feedback eliminated. Aether survives.

### Echo ↔ Feedback
- **Relationship:** Observer vs. subject. Echo watches Feedback's arc.
- **Inferred dynamic:** Echo narrates Feedback's villain arc. Feedback may not be aware of being observed.
- **Missing:** No direct interaction. Echo could challenge Feedback's narrative.

---

## V. The Beatfall Metaphor System

Beatfall maps musical/audio concepts to narrative concepts:

| Audio Concept | Beatfall Meaning |
|--------------|-----------------|
| **Signal** | Truth, clarity, aligned intention |
| **Noise** | Chaos, distortion, misinformation |
| **Rhythm** | Order, timing, fate/destiny |
| **The Drop** | Climax, transformation point |
| **Bass** | Foundation, grounding force (Feedback = bass hacker) |
| **Echo** | Reverberation, delayed response, memory |
| **Aether** | The fifth element (classical), celestial/ethereal quality |
| **Harmony** | Agreement, peace, aligned frequencies |
| **Feedback** | Audio feedback loop — runaway self-amplification |
| **Pitch** | Emotional register, moral alignment |
| **Gain** | Volume, energy, intensity |
| **Tuning** | Adjustment, alignment, course-correction |

**Recommendation:** Create a **metaphor dictionary** that translates musical terms to in-world concepts and use it to generate richer character actions.

---

## VI. The Beatfall Timeline — Observed Events

| Tick | In-World Time | Event | Weather |
|------|--------------|-------|---------|
| 1 | Day 1, 08:01, morning, spring | World initializes. 5 locations. Characters spawn. | clear |
| 1–4 | 08:01–08:04 | Aether: determination. Echo: trust. Feedback: neutral. | clear |
| 4 | 08:04 | Aether first achieves trust. | clear |
| 5–8 | 08:05–08:08 | Aether oscillating determination/trust. Feedback neutral. | clear |
| 10 | 08:10 | Echo happiness begins growing (0.03). | clear |
| 14 | 08:14 | Aether achieves serenity. | clear |
| 16 | 08:16 | Feedback achieves joy (turning point — shadow ascendant). | clear |
| 20 | 08:20 | Weather shifts to partly_cloudy. | partly_cloudy |
| 29 | 08:29 | Feedback achieves ecstasy. Shadow at peak. | partly_cloudy |
| 35 | 08:35 | **Feedback eliminated.** Shadow destroyed at moment of triumph. | cloudy |
| 36–42 | 08:36–08:42 | Aether absent from simulation. | cloudy |
| 40 | 08:40 | Weather shifts to cloudy. | cloudy |
| 43 | 08:43 | Aether returns. Echo + Aether only. | cloudy |
| 60 | 09:00 | Weather shifts to light_rain. | light_rain |
| 80 | 09:20 | Weather shifts to rain. | rain |
| 100 | 09:40 | Weather shifts to **storm.** Aether: joy 0.99, Echo: joy 0.60. | storm |

---

## VII. Open Questions — Unknown Canon

These questions need answers from world-building:

1. **Who/what created Aether, Echo, Feedback?** Are they AI? Spirits? People? Programs?
2. **What is the "Beatfall world"?** Is it a planet? A simulation? A meta-universe?
3. **Why do characters have heartbeat?** Is it metaphorical or literal?
4. **What causes character elimination?** Safety reaching minimum? Energy depleted? Narrative death?
5. **What happens after a 24-hour cycle?** Does the world reset? Does time continue?
6. **Can eliminated characters return?** No evidence either way.
7. **What do Aether's "heroes" look like?** Aether's voice says "Speaks to heroes directly" — are there other heroes Aether is guiding?
8. **Echo's backstory "Memory. Reflection. Reconstruction."** — is Echo actively reconstructing history, or passively recording it?
9. **Feedback's "real-time combat reaction"** — combat with whom? Other characters? The environment? The signal itself?
10. **Is the weather system intentional?** Is someone controlling the storm, or is it emergent from character collective state?

---

## VIII. World-Building Priority Questions

Ranked by narrative importance:

1. **What are the stakes?** What does the world lose if Feedback wins? What does it gain if Aether wins?
2. **Who is the true antagonist?** Is it Feedback, or is Feedback a symptom of something worse?
3. **What is Echo's agency?** Can Echo affect the world, or only observe?
4. **What lies beyond the 5 locations?** Is there a map to explore?
5. **Is there a goal?** Is there an end-state for the simulation, or does it run eternally?

---

## IX. Canon vs. Simulation

Facts established by simulation take precedence over assumptions. If a future run contradicts this document, update the document — the simulation is the ground truth.

Current confidence levels:
- **High confidence:** Character definitions, personality scores, voice lines
- **Medium confidence:** Relationship dynamics (inferred, not explicitly defined)
- **Low confidence:** Metaphor mappings (my interpretation), world lore (incomplete)
- **Unknown:** Everything else — this document is a starting point, not a definitive source
