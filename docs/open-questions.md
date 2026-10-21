# Open questions

The fuzzy parts of the [spec](spec.md). Each one has a suggested answer so a "yes" is enough to close it. When one gets answered, change **Status** to the answer and the date.

Check here first whenever a detail is unclear. Then the spec. Then the [transcript](transcript-2026-10-21.md).

---

### 1. Where does the PoC run?

Several people need to reach it (spec §1), so it can't just live on one laptop.

- **Option A.** A claude.ai artifact. It can store shared results and call Claude without an API key. Testers open a link. Catch: your company may not allow testers on claude.ai.
- **Option B.** A small web app. Closer to the real product, but it needs hosting and an API key.

**Suggested:** A, if company rules allow it. Only synthetic data goes in, so nothing sensitive leaves.
**Status:** open

### 2. Which model, and is there an API key?

Only matters for Option B. This container has no key that I know of.

**Suggested:** use whatever model the CTD tool already uses, so results carry over.
**Status:** open

### 3. Two options per round, or three?

The meeting says both. Two at [1:36], three at [6:07] and [27:39]. The spec picks three.

**Suggested:** three. It's the later and more specific ask.
**Status:** open

### 4. Too many axes for too few rounds

The spec has 8 style axes, 4 rounds, and 1 or 2 axes per round. So each axis gets tested about once. One click per axis is a coin flip, not a preference.

**Suggested:** cut to 4 axes (density, structure, ordering, data handling) and run 6 scored rounds plus 1 repeat round. Every axis then gets tested 3 times. A short mode (4 rounds plus the repeat) tests each axis twice. Details in [build-plan.md](build-plan.md) §4 and §5.
**Status:** open

### 5. Who writes the fact pack?

**Suggested:** I write a synthetic demographics fact pack (made-up trial, made-up numbers). You check that it reads like the real thing.
**Status:** open

### 6. Distiller every round, or once at the end?

Spec §6 says an LLM distiller reruns after every round. That's slower, costs more, and is harder to debug.

**Suggested:** no LLM distiller at all. Count votes per axis after each round, then turn the scores into rules with 16 pre-written sentences. The axes are fixed, so there's nothing for an LLM to discover, and templates can't invent a preference. See [build-plan.md](build-plan.md) §5 and §7.
**Status:** open

### 7. The first tester has no "other person" to compare against

Blind eval option C uses *another* participant's profile (spec Step 5). The very first person has nobody else yet.

**Suggested:** make a fake "opposite" profile for the first run. Real cross-person comparisons start with person 2.
**Status:** open

### 8. Who are the testers, and when?

Spec says 3–5 people, ideally real medical writers. Test readiness is next week, which may eat their time.

**Suggested:** dry run with you and Participant 2 first. Ask for writers after test readiness.
**Status:** open

### 9. Is 70% the right bar?

With 3–5 people, "70%" swings a lot on one person.

**Suggested:** report raw counts ("4 of 5 picked B over A") and skip the percentage.
**Status:** open

### 10. Demographics only?

**Suggested:** yes. One section keeps it small. "Do preferences change by section?" (spec §11) is parked for later.
**Status:** open

### 11. What does Participant 2 want to see at the follow-up?

**Suggested:** a live demo of one person going through the loop, plus a one-page results note.
**Status:** open

### 12. Can we use the CTD tool's real demographics prompt as the baseline?

The blind eval compares the generic prompt (A) against the generic prompt plus the person's style rules (B). If A is the prompt the CTD tool uses today, a win for B means something for the actual product.

**Suggested:** yes, copy the current CTD demographics prompt. If that isn't allowed, use the plain bank prompt in [build-plan.md](build-plan.md) §7.
**Status:** open. Blocks Step 10.

### 13. Who reviews the 48 pre-written texts?

A script checks numbers, length and structure, but someone who writes CSRs should skim all 48 for anything that reads wrong. It takes about 30 minutes.

**Suggested:** one medical writer who is **not** going to be a test participant, so they don't see the texts before their session.
**Status:** open. Blocks Step 7.

---

## Parked (not for the PoC)

- Do preferences change by section type or task? (spec §11)
- Word handoff vs in-tool editor (spec §10.6)
- Steering artifact or golden example (spec §10.4)
