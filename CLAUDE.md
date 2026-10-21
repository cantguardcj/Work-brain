# Work-brain

## Current project: preference-capture PoC

One-line goal: show that a writer picking between three versions of a passage teaches the system their style, and that output written in that style beats a generic prompt.

Read these before working on it:

- `docs/superpowers/plans/2026-10-21-preference-poc.md` is the implementation plan: 14 coding tasks with exact files, function signatures, tests and commands. Build from this.
- `docs/build-plan.md` is the main plan. It's step by step, every step has a "Done when" line, and every design choice cites its research. Section 3 lists where it changes the spec, and section 15 says which skill to use at each step.
- `docs/spec.md` is the original spec. Anything marked (proposed) is not decided yet. Where the build plan changes it, the build plan wins.
- `docs/transcript-2026-10-21.md` is the meeting it came from. If the spec and the meeting disagree, the meeting wins.
- `docs/open-questions.md` lists the decisions that still need a human, a suggested answer for each, and the real answer once decided.

## When a detail is unclear

1. Check `docs/open-questions.md`.
2. Check the spec, then the transcript.
3. Still unclear? Ask the user, then write the answer into `docs/open-questions.md`.

## Skills

`superpowers/skills/` and `pstack/skills/` are copies, not installed. Open the `SKILL.md` for the step you're on, as listed in `docs/build-plan.md` section 15.

## Rules

- Synthetic or de-identified data only. Never real patient or trial data.
- Keep it small. Participant 2 said "this doesn't have to be a big thing."
- Write replies to the user in plain words, in small steps. Run prose through `pstack/skills/unslop`. No em dashes.
