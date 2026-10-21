# Build plan: preference-capture PoC

This is the plan for building the PoC in [spec.md](spec.md). It's written for whoever builds it, whether that's a person, a coding agent pointed at this repo, or both.

Do the steps in order. Every step ends with a **Done when** line. Don't start the next step until that line is true.

Related files:
- [spec.md](spec.md) is the original spec.
- [transcript-2026-10-21.md](transcript-2026-10-21.md) is the meeting. Timestamps like [27:39] point into it.
- [open-questions.md](open-questions.md) holds the decisions that still need a human answer.

---

## 1. The whole thing in one paragraph

A writer clicks through 7 short rounds. Each round shows three versions of the same demographics paragraph, and the versions differ only in style. Every version is written and fact-checked **before** anyone sits down, so nothing is generated live during the rounds. The clicks add up to a score on four style axes. The scores become a few plain rules that get added to the system message. Then the writer blind-ranks three drafts of a new paragraph: the generic prompt, the generic prompt plus their rules, and the generic prompt plus someone else's rules. If people keep picking the draft with their own rules, the idea works.

---

## 2. What we're proving

Participant 2's question at [26:51] was this: "can you capture through people clicking buttons... meaningful preferences that they will appreciate in the output?" That splits into three claims. Each one gets its own test.

| # | Claim | Test | Section |
|---|---|---|---|
| 1 | Clicks capture a real preference, not noise | Round 7 repeats round 2. Does the person pick the same style again? | §9 |
| 2 | Different people want different things | Does each person rank their own rules (B) above someone else's (C)? Do profiles differ? | §9 |
| 3 | Their rules beat the generic prompt | Does each person rank B above A, the generic prompt? | §9 |

If claim 1 fails, claims 2 and 3 don't mean much. That's why the repeat round matters.

---

## 3. Where this plan changes the spec, and why

The spec is a good start. A few parts would make the PoC slower to build or harder to trust. Each change below is backed by a source in §13.

| # | Spec says | Plan does | Why |
|---|---|---|---|
| 1 | 8 style axes | **4 axes** | With 8 axes and about 6 rounds, each axis gets clicked about once. One click is a coin flip. With 4, each axis gets 3 clicks. |
| 2 | Generate candidates live each round | **Pre-generate a bank** of every style combination and fact-check it once | No waiting during sessions. No facts drifting mid-session. Every person sees the same checked texts, so their results can be compared. |
| 3 | Pick next round's axes adaptively | **Fixed, balanced schedule** (§5) | Standard practice in choice research. Six rounds can't support adaptive picking, and a fixed schedule makes people comparable. |
| 4 | LLM "distiller" after every round | **Simple vote counting**, then templated rule sentences | The axes are fixed in advance, so there's nothing for an LLM to discover. Templates can't invent a preference the person never showed. |
| 5 | Show profile summary, then evaluate | **Evaluate first, show the profile after** | If someone reads "you like short text" and then picks, they'll pick the short one because they were told to. That's a primed result, not a real one. |
| 6 | Personalized prompt is its own template | **B = the generic prompt + a style block** | Then A and B differ only by the style block. Any win for B is caused by the preferences and nothing else. |
| 7 | No repeat round | **Round 7 repeats round 2** in a new card order | Checks whether the person's picks are stable. Choice studies use repeat tasks for exactly this. |
| 8 | Eval asks "rank or pick best" | **Pick best, then pick worst** | With three drafts, best plus worst gives a full ranking in two clicks. |
| 9 | One eval passage | **Two eval passages** per person | Doubles the evidence with 3–5 people for about 2 more minutes each. |
| 10 | Success at 70% | **Report raw counts** ("9 of 10") | With 3–5 people, one person moves a percentage by 20 points. |

Changes 2 and 4 together mean the LLM only gets used in two places: building the bank (once, offline) and writing the three eval drafts. That small footprint is the answer to the spec's question "how hard is it to implement?" The plumbing is small. Most of the work is making the bank good.

---

## 4. The four style axes

Each axis has two poles. Every text in the bank sits at one pole on every axis. The pole wording below is what goes into the generation prompt, word for word.

| Axis | Code | Pole A | Pole B | Comes from |
|---|---|---|---|---|
| Density | `D` | "Terse. State facts only. No intro sentence, no transitions, no restating what a table would show. 50–90 words." | "Explanatory. Full sentences that say what the numbers mean, with brief context. 110–160 words." | "short terse factual information and no fluff" [3:31] |
| Structure | `S` | "Prose paragraphs only. No bullets, no tables." | "Bullets or a compact table for the numbers, with one lead-in sentence." | Common split in CSR house styles |
| Ordering | `O` | "Lead with the main takeaway in the first sentence (for example, that the arms were balanced). Details after." | "Walk through the characteristics first. State the main takeaway in the last sentence." | Writer habit; easy to see in one glance |
| Data handling | `H` | "Selective. Report only the 3–4 numbers that matter most and say what they mean." | "Exhaustive. Report every number given for this topic." | The "data dump" complaint [10:12], and CORE Reference's advice against repeating every table number in text |

**Axes dropped from the spec, and why**

- *Sentence length.* Overlaps with Density. Two axes that move together can't be told apart.
- *Terminology.* The system message says "do not change terminology." An axis that changes terminology fights that rule.
- *Voice, hedging.* Real, but weaker signals in a short demographics paragraph. Add them in v2 if the four axes work.

**Length control.** People and LLM judges both lean toward longer text (§13, Singhal; Zheng). Only Density is allowed to change length on purpose. When Density is held fixed, every text must stay within ±15% of the others in word count. The check script in Step 7 enforces this.

---

## 5. Round schedule

**The bank.** Four axes with two poles each make 2⁴ = 16 combinations. Write all 16 for each of 3 training passages, for 48 texts in all.

**The fixed schedule.** Each round tests two axes. Four axes have exactly six pairs, so six rounds cover every pair once and every axis three times.

| Round | Passage | Axes tested | Each axis seen so far |
|---|---|---|---|
| 1 | P1 | D, S | D1 S1 |
| 2 | P2 | O, H | O1 H1 |
| 3 | P3 | D, O | D2 O2 |
| 4 | P1 | S, H | S2 H2 |
| 5 | P2 | D, H | D3 H3 |
| 6 | P3 | S, O | S3 O3 |
| 7 | P2 | O, H (repeat of round 2, new card order) | consistency check |

Round 7 shows the exact same three texts as round 2, shuffled into a new order.

Rounds 1–4 alone test every axis exactly twice. That gives a **short mode** (rounds 1–4, then the repeat round) for a quick demo. It's 5 clicks, close to the "three, four different times" Participant 2 described at [27:39].

**Picking the three cards in a round**

1. Two tested axes have 4 combinations (AA, AB, BA, BB). Drop one at random. Show the other three.
2. Hold the two untested axes at the person's current lean. If an axis has no votes yet, or a tie, pick its pole at random.
3. Shuffle the card order. Log which slot each card was in.

**Scoring**

1. A pick gives one vote, on each tested axis, to the pole the picked card used.
2. **None of these** gives no votes. Log it, and show the reason chips.
3. Axis score = (votes for B − votes for A) ÷ times tested. It runs from −1 (all A) to +1 (all B).
4. Round 7 doesn't vote. It only records whether the person picked the same combination as in round 2.

**Turning scores into rules**

| Votes agreeing (full mode, 3 tests) | Votes agreeing (short mode, 2 tests) | Rule |
|---|---|---|
| 3 of 3 | 2 of 2 | **Firm rule** ("Write tersely.") |
| 2 of 3 | n/a | **Soft rule** ("Lean toward terse writing.") |
| Anything less | 1 of 2 | No rule. Leave this axis out. |

This "needs at least two agreeing signals" rule is the same one pstack's `automate-me` skill uses to decide which habits are real (`pstack/skills/automate-me/SKILL.md`).

**Response time.** Log the milliseconds from cards shown to click. Don't use it for scoring in v1. Research shows faster choices mean stronger preferences (§13, Konovalov & Krajbich), so look at it afterwards in the analysis.

---

## 6. Data shapes

Plain JSON. These refine spec §6.

```jsonc
// One passage of the synthetic fact pack
FactPassage {
  id: "P1",                       // P1–P3 training, E1–E2 eval
  topic: "Age and sex",
  facts: { /* numbers by arm, see Step 2 */ },
  allowedNumbers: ["206", "412", "54.1", "57.8%", ...]  // every number a text may use
}

// One text in the bank
Variant {
  id: "P1-D0S1O0H1",              // passage + pole per axis (0 = A, 1 = B)
  passageId: "P1",
  axes: { D: 0, S: 1, O: 0, H: 1 },
  text: "...",
  wordCount: 74,
  checks: { numbersOk: true, lengthOk: true, structureOk: true, orderingOk: true, humanOk: true }
}

Session {
  id, participantId,              // ID only, never a real name in logs
  mode: "full" | "short",
  startedAt, finishedAt,
  rounds: Round[],
  profile: Profile,
  evals: EvalTrial[],
  workload: { mental, physical, temporal, performance, effort, frustration }  // Raw TLX, 0–100
}

Round {
  n, passageId, testedAxes: ["D", "S"],
  shown: [{ variantId, slot }],   // 3 cards, slot = 0, 1, 2
  pick: variantId | "none",
  reasonChips: string[],
  latencyMs
}

Profile {
  scores: { D: -1..1, S: -1..1, O: -1..1, H: -1..1 },
  timesTested: { D: 3, ... },
  rules: [{ axis: "D", pole: 0, strength: "firm" | "soft", text: "Write tersely." }],
  styleBlock: "...",              // the text added to the system message
  consistent: true | false        // round 7 matched round 2
}

EvalTrial {
  passageId: "E1",
  drafts: [{ label: "A" | "B" | "C", text, slot }],
  best: "A" | "B" | "C",
  worst: "A" | "B" | "C",
  cProfileSource: participantId | "mirror"
}
```

---

## 7. Prompts

**Bank generation** (Step 7, offline)

```
System:
You are a medical writer drafting one part of the "Demographic and Other
Baseline Characteristics" section (ICH E3 section 11.2) of a clinical study
report. Use only the facts given. Do not add, drop, or re-round numbers
except as the style asks. Do not change any terms.

User:
Facts:
{passage facts as JSON}

Write this part of the section in this style:
- {Density pole text}
- {Structure pole text}
- {Ordering pole text}
- {Data handling pole text}
```

**Eval drafts** (Step 10). The user message is identical for A, B and C. Only the system message changes.

```
A  system = {BASELINE}
B  system = {BASELINE} + "\n\n" + {this person's styleBlock}
C  system = {BASELINE} + "\n\n" + {another person's styleBlock}

User (same for all three):
Facts:
{eval passage facts as JSON}
Draft this part of the demographics section.
```

`{BASELINE}` should be the demographics prompt the CTD tool uses today, if you can get it (open question 12). That's the real "generic prompt nobody likes" [15:05], so a win against it means something for the product. If you can't get it, use the bank system message above.

**Style block** (built from the rules, no LLM needed)

```
This writer has these style preferences. Follow them unless they conflict
with the facts.
- {rule text}
- {rule text}
Do not change facts, numbers, or terminology from the source material.
```

Rule sentences come from a fixed lookup. That's 4 axes × 2 poles × 2 strengths = 16 sentences, written once in Step 3.

**Model settings.** Same model, same temperature (0–0.3), same max tokens for A, B and C. Never mention the experiment in any prompt.

---

## 8. Screens

Order matters. The profile is shown **last**, after the blind eval (§3, change 5).

| # | Screen | What's on it | Done when |
|---|---|---|---|
| 1 | Setup (facilitator) | Participant ID, full or short mode, start button | Creates a session record |
| 2 | Round | "Round 2 of 7", three equal cards, all text visible without scrolling on a laptop. **None of these** button. Reason chips appear after the click (optional, skippable). | Click → logged → next round in under 1 second |
| 3 | Blind eval (×2) | Three unlabeled drafts side by side, shuffled. "Which is best?" then "Which is worst?" | Both picks logged; no labels anywhere in the page source |
| 4 | Workload | The six Raw TLX sliders, 0–100 | Logged |
| 5 | Your profile | Plain summary ("You prefer terse prose, main point first"). Thumbs up/down per rule. | Logged. Thumbs don't change this session's results. They feed v2. |
| 6 | Facilitator view | Every person's axis scores side by side, eval results, consistency, None rate, TLX | Exports CSV |

**Reason chips:** "More concise", "Better flow", "Right level of detail", "Too much data", "Wrong order", "Other". These come from the spec, plus "Wrong order" to cover the Ordering axis.

---

## 9. Results: what counts as working

Use counts, not percentages. With 5 people × 2 eval passages there are 10 trials.

| Claim | Measure | Promising | Not shown |
|---|---|---|---|
| 1. Picks are stable | Round 7 matches round 2 | 4 or 5 of 5 people | 2 or fewer of 5 |
| 2. People differ | Pairs of people with opposite firm rules on at least one axis | At least half the pairs | All profiles look the same |
| 2. People differ | Trials where B ranks above C | 8+ of 10 | 6 or fewer of 10 |
| 3. Beats generic | Trials where B ranks above A | 8+ of 10 | 6 or fewer of 10 |
| Axes fit | Rounds answered "None" | Under 20% | Over 30% (the axes miss what people care about) |

**Why 8 of 10?** If B and A were really a coin flip, getting 8 or more of 10 happens about 5% of the time by luck, and 9 or more about 1% of the time. Two caveats. A person's two trials aren't fully independent, and 10 trials is small. Treat this as a signal to keep going, not proof.

For context, published repeat-task agreement in choice studies runs from about 59% (same questions 11 months apart) to 81% (same session). So don't expect 100% on claim 1.

**Before reporting any number, rule out these three things**

1. **Card position.** Count picks per slot. If one slot gets more than half of all picks, people are clicking by position, not style.
2. **Length.** Check whether "always pick the shortest" predicts the picks as well as the profile does. If it does, the finding is "shorter wins", not "personal style wins". That's still worth knowing, but it isn't personalization.
3. **Mirror profile.** For the first person, C uses a flipped copy of their own profile. Check whether B's wins over C hold up once real cross-person C's are in.

---

## 10. Build steps

The skill named on each step is the one to open at that point. Paths are relative to the repo root. "Day" means a working day. Participant 1 is free until next week's test readiness [26:09], so Days 1–2 fall this week. Days 3–5 can slip past test readiness without hurting anything.

### Day 1. Decide and write the inputs

**Step 1. Close the open questions.**
Go through [open-questions.md](open-questions.md) and get a yes or a change on each one. The ones that block building are #1 (where it runs), #2 (model), #12 (baseline prompt) and #13 (who reviews the bank).
Skill: `superpowers/skills/brainstorming`, one question at a time.
**Done when** those four have answers written in the file.

**Step 2. Write the synthetic fact pack.**
Write five passages for a made-up two-arm trial. Make every number internally consistent.

| ID | Topic | Use |
|---|---|---|
| P1 | Age and sex | Training |
| P2 | Race and ethnicity | Training |
| P3 | Baseline disease characteristics | Training |
| E1 | Weight and BMI | Eval |
| E2 | Prior therapy | Eval |

These topics follow what ICH E3 §11.2 covers. The eval passages are topics the person never saw in the rounds, which tests whether the style carries over to new content.

Example of the level of detail (illustrative):
```json
{ "id": "P1", "topic": "Age and sex",
  "arms": { "Drug": { "n": 206, "ageMean": 54.1, "ageSD": 11.0, "female": 119, "femalePct": "57.8%" },
            "Placebo": { "n": 206, "ageMean": 54.6, "ageSD": 11.4, "female": 120, "femalePct": "58.3%" } },
  "total": { "n": 412, "female": 239, "femalePct": "58.0%" } }
```
**Done when** every count adds up across arms and every percentage matches its count.

**Step 3. Write the axis text and rule sentences.**
Copy the pole wording from §4 into a config file. Write the 16 rule sentences (§7).
**Done when** a second person reads each pair of poles and can say which one a sample text matches.

### Day 2. Prototype the screen and lock the shapes

**Step 4. Prototype the round screen.**
A throwaway HTML page with three hand-written cards. Try two or three layouts behind one switcher: side-by-side vs stacked, and chips before vs after the click.
Skills: pstack `prototype` playbook (`pstack/skills/poteto-mode/playbooks/prototype.md`) and `principle-exhaust-the-design-space`.
**Done when** three 160-word cards fit on a 13-inch laptop screen without scrolling, and a layout is picked.

**Step 5. Lock the data shapes.**
Turn §6 into real types in whatever language you're using.
Skills: pstack `architect`, `principle-model-the-domain`.
**Done when** every field in §6 has a home and nothing is stored twice.

**Step 6. Turn this plan into tasks.**
Skill: `superpowers/skills/writing-plans`.
**Done when** every task is small enough to finish and check in one sitting.

### Day 3. Build the bank and the scoring

**Step 7. Generate and check the 48-text bank.**
1. Generate all 16 combinations for P1, P2 and P3 with the bank prompt (§7).
2. Run the check script on every text:
   - **Numbers.** Every number in the text appears in `allowedNumbers`.
   - **Length.** Density A is 50–90 words and Density B is 110–160. Texts that share a Density pole are within ±15% of each other.
   - **Structure.** Structure A has no bullet or table markers. Structure B has them.
   - **Ordering.** Ask the model one yes/no question: "Is the main takeaway in the first sentence?" Ordering A must be yes and Ordering B must be no.
3. Regenerate anything that fails. Log how many tries each one took.
4. A medical writer skims all 48 (about 30 minutes) and flags anything that reads wrong.

Skill: `principle-build-the-lever`. Write the check as a script. Don't eyeball 48 texts.
**Done when** all 48 pass the script and the human skim.

**Step 8. Write the scoring code with tests first.**
Skill: `superpowers/skills/test-driven-development`.
Write these tests before the code:
- All picks at pole A → every score is −1, and every rule is firm, pole A.
- 2 of 3 votes for B → score +0.33, soft rule, pole B.
- "None" in every round → all scores 0 and no rules.
- Short mode, 1 of 2 → no rule.
- Round 7 never changes scores.
- Round 7 matching round 2 → `consistent: true`.
- The style block contains only rules for included axes.

**Done when** all tests pass.

### Day 4. Wire it together

**Step 9. Round flow and logging.**
Screens 1 and 2, the schedule from §5, and an append-only log of every event with a timestamp.
Skill: `superpowers/skills/executing-plans`.
**Done when** a full 7-round run writes a complete session record.

**Step 10. Eval drafts.**
Build A, B and C (§7) for E1 and E2. Run the same number and length checks on them. For the first participant, C uses the mirror profile (every pole flipped).
**Done when** the drafts generate, pass the checks, and are stored with the exact system message that made them.

**Step 11. Remaining screens.**
Eval, then workload, then profile, in that order (§8).
**Done when** a run goes from setup to profile with no dead ends.

### Day 5. Prove it before anyone else sees it

**Step 12. Run it yourself, end to end.**
Skills: `superpowers/skills/verification-before-completion`, `principle-prove-it-works`.
**Done when** you've done one full run and one short run, and both session records look right when you read them by hand.

**Step 13. Dry run with two made-up writers.**
Use an LLM to play two personas. One wants "short, terse, no fluff". The other wants "full explanations, every number". Run each through the loop.
Skill: pstack `eval` playbook (`pstack/skills/poteto-mode/playbooks/eval.md`). Follow its blinding rules.
**Done when** the two profiles come out opposite on at least Density and Data handling.
**Warning:** this only tests the plumbing. Research shows LLM personas collapse toward the same behavior and miss real individual choices, so a pass here says nothing about real writers.

**Step 14. Facilitator view and export.**
**Done when** two sessions show side by side and export to CSV.

**Step 15. Review.**
Skill: `superpowers/skills/requesting-code-review`.
**Done when** every review finding is fixed or has a written reason it isn't.

### Week 2. Real people

**Step 16. Run sessions.**
Do the internal people first (Participant 1, Participant 2), then 3–5 medical writers. Use the script in §11. One person repeats the whole loop a few days later to check stability (spec §8).

**Step 17. Analyze.**
Fill in the §9 table. Run the three checks under it.
Skill: `principle-explain-the-number`.

**Step 18. Results note and meeting.**
Write one page: what we tested, the counts, what surprised us, and what to build next. Then book the follow-up with Participant 2 [25:16].
Skills: pstack `technical-writing`, `unslop`.

### If time runs short

The smallest version that still answers the question is Steps 2, 3, 7, 8, 9 and 10 in short mode, with a plain-HTML eval screen. Skip the facilitator view, the TLX and the reason chips. That version can still show Participant 2 a profile and a blind A/B/C ranking.

---

## 11. Session script (about 20 minutes)

Read this out loud. Don't improvise the intro, because a stray hint can tip people off about what's being tested.

1. **Intro (1 min).** "We're testing a new way to set up drafting preferences. You'll see a few short paragraphs and click the one you'd rather start from. There are no right answers."
   Don't say "personalized", "generic", or "we think your version will win."
2. **Rounds (8–10 min).** Let them click. Don't comment on picks.
3. **Blind eval (4 min).** Two passages. Best, then worst.
4. **Workload (1 min).** The six TLX sliders.
5. **Profile and debrief (4 min).** Show the profile. Ask two questions. "Does this sound like you?" and "Anything missing?" Write down their words.
6. **Close.** Thank them. Don't tell them which draft was theirs until everyone has done the study, so it doesn't spread.

---

## 12. Risks

| Risk | Warning sign | What to do |
|---|---|---|
| People click by position | One card slot gets over half the picks | Already shuffled. Report it if it shows up anyway. |
| "Shorter wins" passes for "style wins" | The shortest-pick rule predicts as well as the profile | Report it honestly. Hold length fixed in v2. |
| Facts drift in a draft | Number check fails | Regenerate. Never show a failed text. |
| Profile summary primes the eval | n/a (prevented) | Profile is shown after the eval. |
| Fatigue | Response times grow round over round. None rate rises late. | Keep passages under 160 words. Use short mode. |
| Too few people | Counts like 5 of 7 | Say so plainly. Call it a signal, not a result. |
| Dry run gives false comfort | Personas pass, people don't | Treat Step 13 as a plumbing test only. |
| Real data leaks in | Any real study name or number | Synthetic data only. Participant IDs, not names. |

---

## 13. Research behind the choices

| Source | What it found | How this plan uses it |
|---|---|---|
| Meissner, Oppewal & Huber, ["How Many Options?"](https://people.duke.edu/~jch8/bio/Papers/HowManyOptions%20Sawtooth%202016.pdf) (Sawtooth 2016), and [Hauser's comments](https://stuff.mit.edu/~hauser/Pages/Hauser%20Comments%20on%20Options%20Sawtooth%202016.pdf) | More options per task gives more information but is harder to answer. Triples became the standard format. | Three cards per round |
| Sawtooth [CBC design guidance](https://info.sawtoothsoftware.com/help/discover/survey-elements/cbc/creating-a-cbc) and [task counts](https://sawtoothsoftware.com/help/lighthouse-studio/manual/cbc-number-of-attributes-levels-tasks.html) | About 6–12 tasks with 3 options each. Many tasks per person is what makes individual-level estimates possible. | 6 scored rounds, fixed balanced design |
| Sawtooth, [holdout and repeat tasks](https://sawtoothsoftware.com/help/lighthouse-studio/manual/holdout-choice-tasks-in-conjoint-analysis-studies.html) | Repeat at least one task to measure how reliable people's choices are | Round 7 repeats round 2 |
| [Liebe et al. 2012](https://link.springer.com/article/10.1007/s10640-012-9567-1); [arXiv 2109.02452](https://arxiv.org/pdf/2109.02452) | Same-choice rates of 59% (11 months apart) and 81% (same session) | Sets realistic expectations for claim 1 |
| Gao et al., [Aligning LLM Agents by Learning Latent Preference from User Edits](https://arxiv.org/pdf/2404.15269) (NeurIPS 2024) | A plain-language description of a user's preference, added to the prompt, cut user edits with no model retraining | Style block in the system message, no fine-tuning |
| Li, Tamkin, Goodman & Andreas, [Eliciting Human Preferences with Language Models](https://proceedings.iclr.cc/paper_files/paper/2025/hash/c9867d5a22653ce98b02595061e40f12-Abstract-Conference.html) (ICLR 2025) | Asking people questions captured what they wanted better than prompts they wrote themselves, and felt like less effort | Supports "clicking instead of writing" [6:07] |
| Kirk et al., [PRISM](https://arxiv.org/html/2404.16019v1) (NeurIPS 2024) | Individuals disagree a lot on subjective text. Feedback should be tied to the person who gave it. | Per-person profiles. Claim 2. |
| Zheng et al., [Judging LLM-as-a-Judge](https://arxiv.org/pdf/2306.05685) (NeurIPS 2023) | Judges favor certain positions and longer answers | Shuffle cards, control length, check slot bias |
| Singhal et al., [A Long Way to Go](https://arxiv.org/abs/2310.03716v2) (COLM 2024) | Much of the apparent preference gain came from answers getting longer | Only Density may change length. "Shortest wins" check. |
| Chiang et al., [Chatbot Arena](https://arxiv.org/pdf/2403.04132) (ICML 2024) | Hide which model wrote what until after the vote | Blind eval, no labels in the page |
| Konovalov & Krajbich, [Revealed strength of preference](https://dlab.sauder.ubc.ca/sjdm/journal/18/181101/jdm181101.html) (JDM 2019) | Faster choices signal stronger preferences and predict whether a choice repeats | Log response time and analyze it later |
| Louviere, Flynn & Marley, [Best-Worst Scaling](https://www.cambridge.org/core/books/abs/bestworst-scaling/introduction-and-overview-of-the-book/20E18DAB89730425DAB7515080121CCC) (Cambridge 2015) | Asking for best and worst gets more information per task | Best + worst in the eval |
| Zhou et al., [IFEval](https://arxiv.org/abs/2311.07911) (2023) | Check instruction-following with rules a script can verify | Scripted checks on the bank |
| Hamilton et al., [CORE Reference](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5794039/) (EMWA/AMWA); [comments and responses](https://emwa.org/resources/the-core-reference-project/comments-and-responses/) | Advises against repeating nearly every table number in CSR text. Lean summaries are fine. | Data handling axis |
| ICH E3, [Structure and Content of Clinical Study Reports](https://www.ema.europa.eu/en/documents/scientific-guideline/ich-e-3-structure-and-content-clinical-study-reports-step-5_en.pdf) | §11.2 covers demographic and baseline characteristics | Defines the test section and passages |
| Hart, [NASA-TLX: 20 Years Later](https://humansystems.arc.nasa.gov/publications/HFES_2006_Paper.pdf) (HFES 2006) | Sensitive, easy workload measure. Changing the questions makes a new, unvalidated instrument. | Raw TLX, all six items, unchanged |
| [Persona simulation limits](https://www.arxiv.org/pdf/2602.15832); [PrivacySIM](https://benchmarklist.com/benchmarks/privacysim/) | LLM personas collapse toward similar behavior. The best model matched real individual choices about 40% of the time. | Dry run tests plumbing only |
| Orne, "On the social psychology of the psychological experiment" (*American Psychologist*, 1962) | People pick up on what an experiment expects and act to match it | Neutral intro script. Profile shown after the eval. |

---

## 14. How this grows into the product

What carries forward to spec §10 if the PoC works:

1. **The profile becomes a per-user setting.** It gets captured once, before any generation, as Participant 2 asked [3:31].
2. **The bank becomes onboarding.** A new writer does the seven rounds on day one.
3. **"Generate three, pick one" becomes a product feature** [6:07]. Each pick keeps updating the profile.
4. **Thumbs up/down on rules** become a cheap way to correct the profile over time.
5. **Next experiment.** Do preferences change by section type? (spec §11). Run the same loop on a second section and compare profiles.

---

## 15. Skills at a glance

`superpowers` runs the process. `pstack` supplies the judgment tools. Use one skill per job and never both packs' version of the same thing.

| Step | Skill | Why there |
|---|---|---|
| 1 | superpowers `brainstorming` | Turns open questions into decisions, one at a time |
| 4 | pstack `prototype`, `principle-exhaust-the-design-space` | Cheap throwaway layouts, compared side by side |
| 5 | pstack `architect`, `principle-model-the-domain` | The data shapes are the experiment. Get them right first. |
| 6 | superpowers `writing-plans` | Small, checkable tasks |
| 7 | `principle-build-the-lever` | A script checks 48 texts better than eyes do |
| 8 | superpowers `test-driven-development` | Scoring bugs would silently ruin every result |
| 9–11 | superpowers `executing-plans` | Work the task list in order |
| 12 | superpowers `verification-before-completion`, `principle-prove-it-works` | Run it for real before calling it done |
| 13 | pstack `eval` playbook | Its blinding rules fit a persona dry run |
| 15 | superpowers `requesting-code-review` | Second look before the demo |
| 17 | `principle-explain-the-number` | Rule out position and length before believing a count |
| 18 | pstack `technical-writing`, `unslop` | A plain one-page results note |
| Any | `principle-laziness-protocol` | Whenever something feels worth adding, check whether it is |

**Left out on purpose:** perf and benchmark tools, formal verification, PR autopilot (`babysit`, `shipping`, `autopilot-*`), and `swarm` / `arena`. None of them fit a small PoC.

Both skill folders are plain copies right now. To use one, open its `SKILL.md`. pstack was written for Cursor, so skip any step that needs a Cursor-only feature (Bugbot, automations, routing work to Grok).
