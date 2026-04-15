# Beatfall — Character Roster

## The Full Cast (22 Characters)

### Heroes (4)
| ID | Name | Archetype | Voice Direction |
|----|------|---------|---------------|
| cadence | Cadence | The Pulse Engine | "Rhythm is everything. Find the beat or make one." |
| riff | Riff | The Sonic Smuggler | "Rules are suggestions. Sound is truth." |
| lyra | Lyra | The Melody Weaver | "Every frequency has a story. I'll find it." |
| resonance | Resonance | The Amplifier | "What's heard isn't what's real. What's felt is what's true." |

### Antagonists (5)
| ID | Name | Archetype | Voice Direction |
|----|------|---------|---------------|
| crescendo | Crescendo | The Escalator | "Louder. Always louder. Until nothing remains but the beat." |
| maestro | Maestro | The Control Freak | "Every frequency serves a purpose. Mine." |
| null | Null | The Void Walker | "The space between sounds is where I live." |
| static | Static | The Scavenger | "I am the noise you can't tune out." |
| vex | Vex | The Audio Terrorist | "Disrupt. Degrade. Derail." |

### Narrator (1)
| ID | Name | Archetype | Voice Direction |
|----|------|---------|---------------|
| tempo | Tempo | The Chronomancer | "I remember how the song began. I know how it ends." |

### Guardians (3)
| ID | Name | Archetype | Voice Direction |
|----|------|---------|---------------|
| overtone | Overtone | The Sage | "Every frequency has a history. Listen long enough." |
| fermata | Fermata | The Paused One | "Rest is not silence. It is anticipation." |
| acoustic | Acoustic | The Forest Spirit | "Some sounds are older than frequency itself." |

### NPCs (6)
| ID | Name | Archetype | Voice Direction |
|----|------|---------|---------------|
| chord | Chord | The Harmonizer | " Alone, I'm noise. Together, I'm music." |
| decibel | Decibel | The Heavy Hitter | "If it hurts to hear, I'm doing it right." |
| pitch | Pitch | The Merchant | "Everything has a price. Name your frequency." |
| bassline | Bassline | The Undercurrent | "You don't hear me. You feel me. That's how it should be." |
| treble | Treble | The Agitator | "High frequencies cut through. That's the point." |
| koda | Koda | The Street Percussionist | "The beat is in the street. I just amplified it." |

### Messengers (2)
| ID | Name | Archetype | Voice Direction |
|----|------|---------|---------------|
| sync | Sync | The Networker | "I'm not the message. I'm the bridge." |
| harmony | Harmony | The Diplomat | "Find the note we all agree on. Build from there." |

### Ambient (1)
| ID | Name | Archetype | Voice Direction |
|----|------|---------|---------------|
| requiem | Requiem | The Death Singer | "Every song ends. Mine just takes longer." |

---

## Echo — The Outlier

Echo exists in the CLI fallback characters but NOT in the seed-expanded character set. Echo's canonical role as "The Main Narrator" conflicts with Tempo's "The Chronomancer." 

**Question for the team:** Which is the canonical narrator? Echo or Tempo?

---

## CLI Fallback vs Seed Characters

The CLI `--chars` flag and `--all` flag load from `FALLBACK_CHARACTERS` (Aether, Echo, Feedback) when the `beatfall/characters` repo is not found.

The **22 seed characters** exist in `packages/store/src/seed-expanded.ts` but are NOT accessible via CLI. They are loaded by the store/world-state system when running as a server.

**For simulation runs via CLI:** Use FALLBACK_CHARACTERS only.
**For full server runs:** Use seed-expanded characters via the server API.

---

## The Three-Character Focus

For narrative clarity, the CEO recommends focusing on ONE TRIO at a time:

**Recommended Trio A (Hero's Journey):**
- Cadence (hero), Crescendo (antagonist), Tempo (narrator)

**Recommended Trio B (Emotional Focus):**
- Resonance (hero), Static (antagonist), Harmony (messenger/narrator)

**Recommended Trio C (Guardian Focus):**
- Overtone (guardian), Null (antagonist), Fermata (guardian)

The current simulation runs use the FALLBACK trio: Aether, Echo, Feedback.

---

*Last updated: April 15, 2026*
*Source: packages/store/src/seed-expanded.ts*
