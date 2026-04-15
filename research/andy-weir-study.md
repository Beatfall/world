# Beatfall Worldbuilding Research — Andy Weir & Science-Driven Narrative

## Why Andy Weir? Core Principles

Andy Weir's approach is uniquely suited to Beatfall's simulation-first world because he treats **constraints as the story engine**. His characters don't have abstract goals — they have specific, measurable problems with specific, measurable solutions.

### The Weir Framework

**1. Every character problem is a problem the READER can understand.**
Mark Watney doesn't just "survive" — he needs to grow potatoes, fix water reclaimer, communicate with Earth. Each problem has: a specific failure mode, a constrained set of resources, and a chain of reasoning from A to B. The reader can FOLLOW the logic.

**2. The character's voice IS the problem-solving process.**
Watney's first-person narration isn't introspection — it's working notes. "I'm going to die if I don't fix X. Option A: requires Y which I don't have. Option B: requires Z which I might be able to make from..." The voice IS the plot.

**3. The world has rules. The character learns them. The problems test those rules.**
In The Martian, NASA and Ares IV have specific engineering constraints. The story doesn't break those rules — it finds the edges of what's possible within them. This creates tension from constraints, not from arbitrary conflict.

**4. Humor is cognitive relief, not distraction.**
Watney jokes when he's figured something out or when the situation is so bad that humor is the only sane response. The jokes mark emotional states — relief, despair, determination — without narrating them explicitly.

**5. The science isn't decoration — it's the CHARACTER.**
Watney is a botanist, so he thinks in biological metaphors. A mechanic would solve the same problems differently. The character's expertise shapes HOW they see and interact with the world.

---

## Mapping Weir to Beatfall — What's Missing

### The Problem: Beatfall Characters Don't Have Problems (They Have States)

In our v3 simulation, characters have moods and happiness scores, but:
- **Aether** goes from determination → joy → serenity → ecstasy based on... the weather? Group dynamics? There's no specific problem being solved.
- **Echo** observes mood states but has no narrative function beyond "witness"
- **Feedback** was suppressed before it could articulate WHY it wanted to break things

In Andy Weir, every character's state is the DIRECT RESULT of the problem they're solving right now. Mark Watney isn't "a determined person" — he's a guy trying to make water out of hydrazine with limited oxygen.

### What Beatfall Should Have (Weir-Style)

Each character needs:

**A Specific Technical Problem:**
- Aether's problem: "How do I stabilize the signal when Feedback is corrupting the frequency?"
- Echo's problem: "How do I record accurately when Feedback is introducing static/distortion?"
- Feedback's problem: "How do I increase gain when the system keeps trying to suppress me?"

**Resource Constraints:**
- Aether: limited energy, can only maintain clarity for N ticks before needing to recharge
- Echo: limited observation bandwidth (can only track 2 characters simultaneously?)
- Feedback: limited gain (increasing gain increases visibility, which attracts suppression)

**A Chain of Reasoning:**
Each tick shouldn't just change mood — it should advance or resolve part of the problem chain. "Aether tried to tune Feedback out but the storm is making it harder to hear."

### The Missing Element: Character-Specific Problem Chains

**Aether's Problem Chain (Andy Weir Style):**
```
Tick 1: I'm on a frequency. Feedback is going to try to corrupt it.
Tick 5: I can maintain clarity at current energy level. But Feedback is getting louder.
Tick 10: If I don't do something, I'll lose the signal. Option: filter. Option: amplify.
Tick 15: I amplified but that made Feedback louder too. The coupling is worse.
Tick 20: Storm is making the medium unstable. I'm fighting both Feedback AND the weather.
Tick 25: I need a new strategy. Echo might be able to record the pattern and help me predict Feedback.
```

**Feedback's Problem Chain (Andy Weir Style):**
```
Tick 1: My natural state is distortion. I should be able to increase gain.
Tick 5: But the system keeps damping me. Why? Aether's clarity is suppressing me.
Tick 10: If I can't increase gain directly, I'll try to disrupt Aether's signal first.
Tick 15: Disruption worked — Aether's clarity dropped. Now I can try to increase gain again.
Tick 20: But Echo is recording everything. If Echo shows Aether the pattern, I'll lose the element of surprise.
Tick 25: I need to also corrupt Echo's recording. But that requires more gain than I have.
```

This creates NARRATIVE from PROBLEMS instead of from MOOD TRANSITIONS.

---

## Andy Weir's "Show the Work" Applied to Beatfall

### The Core Technique: Make the Reader/Camera See What the Character is Doing

Andy Weir's famous first-draft technique: write out the ENTIRE process of solving the problem, step by step, including the dead ends. Then strip out everything the reader can infer.

For Beatfall simulation, this means:

**Instead of:** "Aether achieved serenity at tick 14"
**We should see:** "Aether found a quiet frequency at tick 14. The static from Feedback was still audible, but at -40dB — below the threshold where it corrupts the signal. Aether tested: could they send a message? Yes. The clarity held for 5 minutes. Aether marked the frequency as STABLE."

The difference: `serenity` is a MOOD. `STABLE` is a CONDITION with CONSEQUENCES.

### Implementing This in Simulation

The mental model needs a `problemState` field per character:

```
Aether.problemState: {
  currentProblem: "maintain-signal-clarity",
  strategy: "frequency-isolation",
  resources: { energy: 0.7, clarity: 0.8 },
  recentAction: "stabilized-frequency",
  lastResult: "success",
  nextMilestone: "reach-tick-50-without-corruption",
  deadEnds: ["filter-attempt-1", "amplify-attempt-1"],
}
```

This feeds into the LLM prompt (when running --live) or the narrative description (in --mock mode).

---

## Other Authors' Techniques for Beatfall

### Brandon Sanderson — Magic System Rules

Sanderson's Three Laws of Magic applied to Beatfall's audio-signal system:

**1. The limitations matter more than the powers.**
Aether can "bring clarity" but:
- Clarity requires energy (depletes with use)
- Clarity has a range (nearby characters only)
- Clarity can be overcome by sufficient gain (Feedback's distortion)
- Clarity has a frequency (one signal at a time)

**2. The magic is not the story.**
Aether doesn't win because of clarity — Aether wins because of HOW they use clarity in response to specific problems.

**3. The magic costs something.**
Every time Aether brings clarity:
- Energy depletes
- The signal becomes more visible (easier for Feedback to target)
- Echo's recordings get clearer (but Feedback can also listen in)

### Dan Harmon — Character Want vs. Need

Harmon's 4-quadrant character arcs (applied to Beatfall):

**Want:** What the character thinks they want
- Aether wants Feedback eliminated
- Echo wants an accurate record
- Feedback wants to break the signal

**Need:** What they actually need
- Aether needs to learn that suppressing Feedback doesn't work — only harmonizing does
- Echo needs to realize that its recordings shape the characters, not just observe them
- Feedback needs... to be heard? To stop being painful? To become part of the harmony?

**The Lie They Believe:** 
- Aether: "If I stay perfectly clear, nothing can break me"
- Echo: "If I record everything perfectly, I don't need to act"
- Feedback: "If I break the signal loud enough, I won't have to feel the consequences"

### Ursula K. Le Guin — The World Has a History

Le Guin's Earthsea: names have power, the world remembers. For Beatfall:

- The signal has a HISTORY. What happened on this frequency before?
- Echo's recordings ARE the history. But history inverts the present — old recordings show what the signal USED to be.
- The weather system is memory made weather. The storm at tick 100 isn't random — it's the world REMEMBERING the last time Feedback was suppressed.

### Brandon Sanderson — Failure Is the Unit of Story

Sanderson on what makes climaxes satisfying: "The character tries everything. Everything fails. Then they do the one thing no one expected, including themselves."

For Beatfall: The Act III climax should be Aether trying everything to eliminate Feedback, everything failing, and then Aether doing the UNEXPECTED thing — inviting Feedback into the harmony instead of fighting it.

---

## Practical Implementation — Beatfall v4 Problem-First Design

### New Character Structure

Each character gets these additional fields (beyond personality/empathy/sociability):

**`problemState`:**
```typescript
interface ProblemState {
  name: string;           // Short name: "frequency-isolation"
  phase: 'analysis' | 'attempt' | 'failure' | 'partial-success' | 'success';
  attempts: number;       // How many times tried this approach
  resources: Record<string, number>;  // energy, clarity, gain, etc.
  deadEnds: string[];     // Failed strategies (by name)
  nextExpected: string;   // "If this works, then X should happen"
}
```

**`voicePurpose`:**
```typescript
// What the character's voice is FOR (in Andy Weir terms)
type VoicePurpose = 
  | 'problem-solving-log'   // Aether: thinking through options
  | 'historical-record'      // Echo: documenting what happened
  | 'pressure-escalation';   // Feedback: marking urgency
```

### Problem Chain Rules

1. **No character can stay in the same problemState for more than 20 ticks** — must either progress or fail
2. **Every failure must have a visible cause** — not "I failed" but "I failed because I used X which was depleted"
3. **Every success must have a cost** — not "I solved it" but "I solved it but now Y is depleted"
4. **Dead ends must be named** — not "that didn't work" but "Strategy 'filter' failed because Feedback learned to ride above the filter frequency"

### Simulation Modifications Needed

The `run-simulation.js` parser should be extended to capture problem-state transitions (from log messages), not just mood transitions.

The Dashboard should display CURRENT PROBLEM per character alongside mood/energy/happiness.

The `MentalModelEngine.tick()` should advance problem state, not just apply mood decay.

### Recommended Reading for Beatfall Team

1. **Andy Weir's original blog** (andyweirauthor.com) — he posted The Martian chapter by chapter, with commentary on the science problem-solving process
2. **"The Martian"** — the canonical example of problem-driven narrative
3. **"Project Hail Mary"** — Weir's sequel, where the problem becomes even more abstract (what is memory? what is identity?)
4. **Brandon Sanderson's YouTube lectures** — "Sanderson's Laws of Magic" + his BYU writing classes (free on YouTube)
5. **Dan Harmon's "Story Structure" talks** — the 4-quadrant arc and the "wants vs. needs" framework (his Reddit posts on story structure are gold)
6. **Le Guin's "The Language of the Night"** (essays on fantasy/sf) — on why worldbuilding should serve story, not replace it

---

## Beatfall-Specific: The Signal/Noise Metaphor as Andy Weir Problem

Andy Weir's stories are about SURVIVAL IN HOSTILE CONDITIONS using TECHNICALLY CONSTRAINED SOLUTIONS. Beatfall's audio-signal metaphor is PERFECT for this framework:

**The Core Problem (Andy Weir Style):**
- A signal exists in a noisy environment
- Three characters are on the same signal but with different hearing
- Aether wants to preserve the signal
- Echo wants to record the signal accurately
- Feedback wants to distort the signal
- The WEATHER is the medium — storms corrupt, calm preserves

**The Problem Trees:**

Aether's problems:
1. "How do I filter out Feedback's distortion?" → requires energy
2. "How do I warn Echo without revealing my position to Feedback?" → requires strategy  
3. "How do I survive the storm at tick 100?" → requires planning ahead

Feedback's problems:
1. "How do I increase gain when Aether's clarity keeps damping me?" → requires timing
2. "How do I corrupt the signal when Echo is recording everything?" → requires speed
3. "How do I break through when the storm (weather) is making everyone defensive?" → requires environment

Echo's problems:
1. "How do I record accurately when Feedback's distortion is interfering?" → requires selection
2. "Should I warn Aether about Feedback's approach at the cost of revealing my position?" → requires ethics
3. "How do I preserve the record when the storm might destroy the recording medium?" → requires backup

---

## Next Steps for Implementation

### Phase 1: Add Problem State to Characters (simulation-level)
- Extend `CharacterInstance` with `problemState` field
- MentalModelEngine generates problem state changes based on environment
- Dashboard shows current problem for each character

### Phase 2: Make Mood a Consequence, Not a Cause (narrative-level)  
- Mood should change AS A RESULT of problem state, not independently
- "Aether is determined" should mean "Aether is in the middle of solving problem X and hasn't failed yet"
- "Aether is ecstatic" should mean "Aether just solved a major problem and the solution worked"

### Phase 3: Connect to StorylineDirector (director-level)
- StorylineDirector beats should fire based on problem state, not mood
- "Problem-failure beat": character fails a strategy → mood drops → new problem
- "Problem-solving beat": character solves a problem → mood rises → new, harder problem

### Phase 4: Run Simulation and Verify
- Run 200-tick simulation with problem state enabled
- Verify that mood transitions are CAUSED by problem events, not random
- Verify that Feedback's arc is a PROBLEM CHAIN, not a mood arc
