# Preference-capture PoC spec

> Saved as given on 2026-10-21. Items marked **(proposed)** are suggestions, not decisions. Fuzzy details and their answers are tracked in [open-questions.md](open-questions.md). The meeting this came from is in [transcript-2026-10-21.md](transcript-2026-10-21.md).

This spec covers the **preference-capture proof of concept** Participant 2 asked Participant 1 to mock up. It then covers how the PoC would grow into the product described in the meeting. Items from the meeting are cited. Everything else is my proposed design, and I've labeled it **(proposed)**.

#### 1. Purpose and hypothesis

**Goal:** prove that clicking buttons can capture a writer's style preferences, and that embedding those preferences in generation produces output the writer likes more than a generic prompt does.

**Hypothesis under test:** a generic prompt is the average of all opinions, and different writers (e.g., Jang vs. Rebecca) will pick different outputs.

**Scope rule:** it should be small. The PoC should support multiple people and multiple generated outputs.

**Success looks like:**
* The PoC captures preferences and uses them in the output.
* Evaluators prefer personalized output over generic output.
* Two different people end up with visibly different profiles.

#### 2. Scope

| In scope | Out of scope (Phase 1) |
|---|---|
| Preference-capture loop (pick 1 of 3, or none) | In-tool document editor |
| Preference profile → system message | Evidence mapping automation |
| Generation of a test passage with the profile | Agentic long-running section builds |
| Multi-user sessions and comparison view | Word add-in or export polish |
| Logging and evaluation | Production auth, permissions, audit trail |

#### 3. Actors and setup

* **Participant (writer or tester):** goes through the loop and rates results.
* **Facilitator (Participant 1):** configures the session and reviews logs.
* **Test content (proposed):** use a single fixed **source fact pack** so only style varies. Participant 1 suggested a demographics section as the test case. Use synthetic or de-identified data only.

#### 4. Process flow, step by step

**Step 0: Session creation**
1. The facilitator picks the section type (demographics) and the fact pack.
2. The facilitator enters the participant name or ID, plus the number of rounds (default 4, configurable 3–6). Participant 2 described clicking "three, four different times."
3. The system creates a session record with a unique ID and a timestamp.

**Step 1: Baseline generation (proposed)**
1. Generate one **generic-prompt output** for a held-out passage and store it.
2. Hide it from the participant until the evaluation step. This is the control.

**Step 2: Preference rounds (repeat N times)**

For each round:
1. **Candidate generation (proposed):**
   * The system generates **three** versions of the same content.
   * Each version differs along one or two **style axes** (see §5).
   * Content and facts stay identical across the three versions.
2. **Display:** show the three versions as large clickable cards, in randomized order.
3. **Participant action:** click **one**, or click **None of these**.
4. **Optional reason chips (proposed):** after the click, show one-tap tags such as "More concise," "Better flow," "Right terminology," and "Too much data." This captures the "why" Participant 1 wanted, without making the writer type.
5. **Logging:** store the three texts, the axis values for each, the pick, the reason chips, and the response time.
6. **Profile update:** the preference engine recomputes the profile (see §6).
7. **Next-round selection (proposed):** pick axes where the profile is least certain.
8. **"None of these" handling:** log it as a rejection of all three axis combinations. The next round must move away from them.

**Step 3: Profile finalization**
1. After round N, the system produces the final **preference profile** (JSON plus a natural-language style paragraph).
2. The participant sees a plain-language summary, e.g., "You prefer short, factual sentences, no introductory framing, and numbers in prose."
3. Optional: the participant can thumbs-down individual rules. This is a cheap correction.

**Step 4: Personalized generation**
1. The profile paragraph is **embedded in the system message** for every later generation call.
2. Generate output for the held-out passage using the profile.

**Step 5: Blind evaluation (proposed)**
1. Show the participant three unlabeled outputs for the same passage:
   * **A:** generic prompt (baseline)
   * **B:** personalized with *their* profile
   * **C:** personalized with *another participant's* profile (cross-person control)
2. The participant ranks them, or picks the best.
3. Record the result and the reasons.

#### 5. Style axes (proposed)

| Axis | Pole A | Pole B |
|---|---|---|
| Density | Terse, factual, no fluff | Narrative, explanatory |
| Sentence length | Short | Long, compound |
| Structure | Prose | Bulleted or tabular |
| Ordering | Conclusion first | Build-up to the conclusion |
| Hedging | Direct statements | Qualified language |
| Voice | Active | Passive |
| Data handling | Selective, interpreted | Exhaustive, listed |
| Terminology | Spelled-out terms | Abbreviations |

The "terse, no fluff" pole comes straight from Participant 2's example of a writer's likely preference. The "data handling" axis targets the "data dump" complaint.

#### 6. Preference engine (proposed)

**Data model**
```
Session { id, participantId, sectionType, factPackId, rounds[], profile, evalResults }
Round { n, candidates[3] {text, axisValues}, pickIndex | null, reasonTags[], latencyMs }
Profile { axisScores{axis: -1..+1, confidence}, rules[], styleParagraph, version }
```

**Update logic**
1. Each pick adds evidence toward the picked version's axis values and against the others.
2. "None" adds negative evidence for all three.
3. After each round, an LLM **distiller** reads the full round history and outputs updated axis scores, confidence, and 3–7 explicit rules.
4. Rules with low confidence are marked tentative. They are excluded from the final system message, or phrased softly.

**System-message template**
```
You are drafting [section type] for a clinical document.
Follow this writer's style preferences:
{styleParagraph}
Hard rules: {rules}
Do not change facts, numbers, or terminology from the source material.
```

#### 7. UI spec (proposed)

* **Screen 1, Setup:** section, fact pack, participant name, round count.
* **Screen 2, Round:** a progress indicator ("Round 2 of 4"). Three equal cards, with the full text visible without scrolling. Below them, a **None of these** button and optional reason chips. No typing required.
* **Screen 3, Profile review:** the plain-language summary, with a per-rule thumbs up or down.
* **Screen 4, Evaluation:** three blind outputs side by side, with a rank-or-pick control.
* **Screen 5, Facilitator comparison:** profiles for multiple participants side by side, plus the evaluation results.

The design principle is to move people from writing to clicking.

#### 8. Evaluation plan (proposed)

* **Participants:** at least 3–5 people, ideally including real medical writers.
* **Primary metric:** how often personalized output (B) beats generic output (A). The threshold is yours to set. I'd start at 70% or higher.
* **Differentiation metric:** whether each person prefers their own profile's output (B) over someone else's (C). This tests whether preferences differ meaningfully.
* **Efficiency metrics:** time to complete the loop, rate of "None" picks, and reason-chip distribution.
* **Stability check:** rerun the loop for one person on a different day. Profiles should be broadly consistent.

#### 9. Risks and edge cases

* **Inconsistent picks:** if a person contradicts themselves, lower the confidence rather than flip the rule.
* **Axis confounding:** vary only one or two axes per round so a pick can be attributed.
* **Content drift:** validate that every candidate keeps the same facts and numbers. Run an automated fact-check pass before display.
* **Small sample:** 3–5 rounds may not capture everything. This is expected for a PoC.
* **Sensitive data:** use synthetic or de-identified fact packs only.

#### 10. Phase 2: how this fits the full product

These pieces come from the broader discussion. None are in the PoC.

1. **Preference profile as a persistent per-user setting.** Capture it before any generation occurs.
2. **Plan-then-go workflow.** The writer describes their goal, the system produces a plan they can inspect, then they click a button and review the output.
3. **First-pass mapping.** Participant 2 thinks a coherent first pass is very likely achievable, and writers dislike mapping most.
4. **Steering artifact.** Nate suggested a template or golden example as the agent's input. What belongs in it, and where the agent should stop, are open questions. Participant 1 thinks this would fix much of the continuity problem. Participant 2 doubts it scales to every section of every document.
5. **Boundary detection.** The agent needs to detect when to stop, go deeper, or go shallower.
6. **Word handoff.** This is unresolved. Participant 2 wants time in the tool near zero and no in-tool editor. Participant 1 expects some writers to want full control in the tool. A reasonable default is a high-quality Word export, with an editor as a later experiment.

#### 11. Open questions to resolve first

* How hard is it to implement preference capture and embedding?
* Will click-based picks reveal preferences writers actually appreciate in the output? Participant 2 called this "still a theory."
* How many rounds are enough?
* Do preferences vary by section type or task, or only by person?
* Does the PoC belong in Participant 1's scope? They raised this concern themselves.

#### 12. Suggested timeline (proposed)

Participant 1 expects to have no real responsibilities until next week, and the team is wrapping up template work before test readiness.

1. **Days 1–2:** write the fact pack, axes, and generation prompts. Build the round UI with static mock candidates.
2. **Days 3–4:** wire up live generation and the distiller. Add logging.
3. **Day 5:** add the evaluation screen. Run an internal dry run.
4. **Follow-up:** Participant 1 plans to book a meeting with Participant 2 to review the A/B approach. Do that after the dry run.
