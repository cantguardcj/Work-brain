# Skills plan

Which skill I pick up at each step of the preference PoC, and why. The PoC itself is in [spec.md](spec.md). The fuzzy parts live in [open-questions.md](open-questions.md).

## The short version

- **superpowers runs the process.** Brainstorm, write a plan, work the plan, check it, finish the branch.
- **pstack supplies the judgment tools.** Prototype cheaply, sketch the data shapes, run a blind eval, keep the writing clean.
- **One skill per job.** Both packs have a TDD skill and a planning skill. I use one of each, never both, so no step gets done twice.

## Before any of this works

Right now the skills are plain folders in this repo. Claude Code won't load them on its own. Two ways to fix that:

1. I open the right `SKILL.md` by hand at each step below. Works today with no setup.
2. I link only the skills this plan uses into `.claude/skills/` so they load by themselves.

I'd pick option 2, but only for the chosen ones. Loading all ~80 buries the useful ones. Also, superpowers' `using-superpowers` skill makes the agent check for skills before every single reply, which gets noisy fast. I'd leave that one out.

pstack was written for Cursor. A few steps assume Cursor-only features (Bugbot, automations, sending work to Grok). In Claude Code those steps just get skipped. That's fine for this project.

## Phase 0. Pin down the fuzzy parts (today)

| # | Skill | When | Why |
|---|---|---|---|
| 1 | pstack `in-your-own-words` | First thing | I say back what I think the goal is. If I've got it wrong, we find out in one message, not after a day of building. |
| 2 | superpowers `brainstorming` | Right after | Walks the open questions one at a time, each with a suggested answer. Most of the spec says "(proposed)". This is where proposed turns into decided. |
| 3 | none | After each answer | Write the answer into `open-questions.md` so nobody asks it twice. |

## Phase 1. Shape it (days 1–2)

| # | Skill | When | Why |
|---|---|---|---|
| 4 | pstack `prototype` playbook | Before real code | The spec says build the round screen with fake candidates first. This playbook is exactly that. Throwaway HTML, no tests, decide the layout fast. |
| 5 | `principle-exhaust-the-design-space` | Inside step 4 | Build 2 or 3 versions of the pick screen behind one switcher (cards side by side vs stacked, chips before vs after the click). Choose by looking, not guessing. |
| 6 | pstack `architect` + `principle-model-the-domain` | After the layout is picked | Sketch the Session, Round and Profile shapes from spec §6 before writing logic. Those shapes are the experiment. If they're wrong, the logs can't answer the questions we care about. |
| 7 | superpowers `writing-plans` | After step 6 | Turn the spec into a list of small build tasks, each one checkable on its own. That list drives days 3–4. |
| 8 | none | Alongside step 7 | Write the synthetic demographics fact pack. No skill needed, just care with the numbers. |

## Phase 2. Build (days 3–4)

| # | Skill | When | Why |
|---|---|---|---|
| 9 | superpowers `executing-plans` | The whole phase | I work the plan task by task, in this session. The alternative, `subagent-driven-development`, spins up extra agents. Overkill for something this small. |
| 10 | superpowers `test-driven-development` | Preference engine only | The code that turns clicks into axis scores. If it's wrong, the profiles come out wrong and nobody notices until the eval. The screens don't need tests. |
| 11 | `principle-build-the-lever` | Content drift check | Spec §9 says all three candidates must keep the same facts and numbers. Instead of eyeballing, I write a small script that pulls the numbers out of each candidate and compares them to the fact pack. |
| 12 | `principle-laziness-protocol` | Whenever I want to add something | Participant 2 said "this doesn't have to be a big thing." This is the brake. |
| 13 | superpowers `systematic-debugging` | Only when something breaks | Find the cause before touching code. |
| 14 | pstack `show-me-your-work` | Every design choice | A plain log, one line per choice (what, why, evidence). So much of the spec is "proposed" that you'll want to see which way I went without rereading the chat. |

## Phase 3. Prove it (day 5)

| # | Skill | When | Why |
|---|---|---|---|
| 15 | superpowers `verification-before-completion` + `principle-prove-it-works` | Before saying "done" | I run the whole loop myself as a fake participant, start to finish. "It compiles" doesn't count. |
| 16 | pstack `eval` playbook | Dry run | Two made-up writers. One wants short and terse, the other wants long and narrative. Their profiles should come out clearly different. The playbook's blinding rules also line up with spec Step 5 (no labels, random order). This only tests the plumbing. Real writers still have to judge real taste. |
| 17 | `principle-explain-the-number` | Before reporting any result | Before anyone says "personalized won 80% of the time", check how many picks that is, and whether card position or text length explains it better than style. |
| 18 | superpowers `requesting-code-review` | Before the demo | A second look before Participant 2 sees it. pstack's `interrogate` does the same with several models, but it leans on Cursor, so I'd skip it here. |

## Phase 4. Hand it off

| # | Skill | When | Why |
|---|---|---|---|
| 19 | pstack `technical-writing` + `unslop` | Before the follow-up meeting | A one-page results note for Participant 2. Short, plain, no AI tells. |
| 20 | superpowers `finishing-a-development-branch` | After the meeting | Decide how this branch lands. PR, merge, or park it. |
| 21 | pstack `ketchup` | Any time you come back after a break | A short "here's what changed, here's what's waiting on you." |

## Skills I'm leaving out

- **Speed tools** (`benchmark-checklist`, `hillclimb`, perf playbooks). Nothing here needs to be fast.
- **Formal verification skills.** Far too heavy for a PoC.
- **PR autopilot** (`babysit`, `shipping`, `autopilot-*`). No CI yet. Nothing to babysit.
- **`swarm` and `arena`.** They launch many agents at once. Costly, and you haven't asked for that.
- **`typescript-best-practices`.** Only comes back in if we end up writing TypeScript.

## One idea worth stealing

pstack's `automate-me` is a tiny version of this whole PoC. It reads someone's past chats, finds their habits, and writes them into a reusable instruction. Its confidence rule is simple. A habit seen in 2 or more slices counts. A habit seen once gets dropped.

That's a good rule for spec §6 step 4 (which style rules count as tentative). I'd use the same idea. A style rule needs at least 2 agreeing picks before it goes into the system message.
