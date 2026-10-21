# Preference-Capture PoC Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking. The skills live in `superpowers/skills/` in this repo.

**Goal:** A small web app where a writer clicks through 7 rounds of "pick the paragraph you'd rather start from", gets a style profile, and blind-ranks three drafts (generic prompt, generic + their style, generic + someone else's style).

**Architecture:** All decisions live in pure TypeScript modules under `src/core/` (fact pack, checks, schedule, scoring, profile, analysis). An offline script pre-writes and checks a bank of 48 paragraphs. A small Express server runs sessions, stores them as JSON files, and calls the LLM only to write the three eval drafts. A Vite-built vanilla TypeScript client shows the screens.

**Tech Stack:** Node ≥ 20, TypeScript (strict), Express 4, Vite 5, Vitest, Playwright, supertest, `@anthropic-ai/sdk`, `csv-parse` / `csv-stringify`, `tsx`.

**Spec:** `docs/build-plan.md` (main) and `docs/spec.md` (original). The transcript is `docs/transcript-2026-10-21.md`.

**Assumptions to confirm** (see `docs/open-questions.md`):
- Q1 is answered as "small web app". If a claude.ai artifact is chosen instead, Tasks 1–7 and the `analysis.ts` part of Task 12 carry over unchanged. Tasks 8–11 change.
- Q2 is answered as "Anthropic API, model from the `LLM_MODEL` env var". Any provider works if it implements the `LlmClient` interface in Task 6.
- Q12 (baseline prompt) defaults to `content/baseline-prompt.txt`, holding the bank system prompt until the real CTD prompt is pasted in.

## Global Constraints

- All code lives in `poc/`. Every command runs from `poc/`.
- Synthetic data only. Fictional study `XR17-301`, fictional drug `XR-17` vs placebo, in moderate-to-severe plaque psoriasis. Arms are 206 and 206, total 412, in every passage.
- `participantId` must match `/^[A-Za-z0-9-]{1,20}$/` (IDs like `W01`, never names).
- Exactly 3 cards per round. The round schedule is exactly the one in Task 3.
- Density word bands are 50–90 (pole 0) and 110–160 (pole 1). Spread is within ±15% of the group mean.
- A, B and C drafts use the same model, `temperature: 0.2` and `max_tokens: 600`. Only the system prompt differs.
- No prompt text written by us contains any of: `experiment`, `personaliz`, `personalis`, `generic`, `candidate`, `judge`, `rubric`, `variant`, `A/B`.
- Before both eval choices are recorded, the participant never sees draft labels, system prompts, variant IDs, or their profile.
- Reason chips are exactly `More concise`, `Better flow`, `Right level of detail`, `Too much data`, `Wrong order`, `Other`.
- Workload is Raw NASA-TLX: the six standard items, 0–100, standard wording, unchanged.
- Screen copy is exactly as written in Task 11.

## Review Focus

1. **Refresh or double-click mid-round.** The participant lands back on the same round and the pick counts once. Tests are in Task 9 (`resumes after restart`, `same pick twice is a no-op`) and Task 11 (`reload keeps the round`).
2. **LLM errors while eval drafts are prepared.** Each draft retries up to 3 times. If it still fails, the session shows "Something went wrong. Please tell the facilitator." and the facilitator can retry. No data is lost. Tests are in Task 8 (`marks evals failed`) and Task 9 (`retryEvals recovers`).
3. **A person with no clear rules.** Their B prompt equals A. The trial is flagged `bEqualsA` and left out of the B-over-A count. Tests are in Task 5 (`empty style block`), Task 8 (`flags bEqualsA`) and Task 12 (`excludes bEqualsA`).
4. **Labels leaking to the participant.** No API response before the profile step contains `label`, `system`, or a variant ID, and the profile endpoint refuses until both eval choices are in. Tests are in Task 10.
5. **First participant, or everyone has the same profile.** C falls back to a mirror of the participant's own profile, and the source is recorded as `mirror`. Tests are in Task 8 (`chooseCProfile`).

---

## File Structure

```
poc/
  package.json, tsconfig.json, vite.config.ts, playwright.config.ts, .gitignore
  content/
    fact-pack.json         synthetic facts, 5 passages (Task 1)
    axes.json              pole wording, rule sentences, summary phrases (Task 5)
    baseline-prompt.txt    system prompt for draft A (Task 8)
    bank.json              48 generated paragraphs (Task 7 output)
    bank-review.csv        human review sheet (Task 7 output)
  src/core/
    types.ts               every shared type (Task 1)
    factPack.ts            load, validate, allowed numbers (Task 1)
    checks.ts              number, length, structure checks (Task 2)
    rng.ts                 seeded RNG and shuffle (Task 3)
    schedule.ts            round schedule, card picking, variant IDs (Task 3)
    scoring.ts             votes, scores, rules, consistency (Task 4)
    profile.ts             rule text, style block, summary, mirror (Task 5)
    analysis.ts            results counts and CSV (Task 12)
  src/llm/
    client.ts              LlmClient, Anthropic client, fake client (Task 6)
    prompts.ts             every prompt we send (Task 6)
  src/bank/
    generate.ts            write and check bank paragraphs (Task 7)
    review.ts              review CSV, bank readiness (Task 7)
  src/server/
    errors.ts, store.ts, evalDrafts.ts, engine.ts, app.ts, index.ts (Tasks 8–10, 12)
  src/client/
    index.html, main.ts, api.ts, style.css, screens/*.ts (Tasks 11–12)
  src/scripts/
    bank-cli.ts, llm-smoke.ts, persona-run.ts (Tasks 6, 7, 13)
  tests/
    core/*.test.ts, llm/*.test.ts, bank/*.test.ts, server/*.test.ts
    fixtures/pack.ts, fixtures/bank.ts
    e2e/session.spec.ts
```

---

### Task 1: Project scaffold, shared types, fact pack

**Files:**
- Create: `poc/package.json`, `poc/tsconfig.json`, `poc/.gitignore`
- Create: `poc/src/core/types.ts`, `poc/src/core/factPack.ts`
- Create: `poc/content/fact-pack.json`
- Test: `poc/tests/core/factPack.test.ts`, fixture `poc/tests/fixtures/pack.ts`

**Interfaces:**
- Produces, in `src/core/types.ts` (every later task imports from here):

```ts
export type Axis = 'D' | 'S' | 'O' | 'H';
export const AXES: readonly Axis[] = ['D', 'S', 'O', 'H'];
export type Pole = 0 | 1;
export type AxisCombo = Record<Axis, Pole>;
export type TrainingPassageId = 'P1' | 'P2' | 'P3';
export type EvalPassageId = 'E1' | 'E2';
export type PassageId = TrainingPassageId | EvalPassageId;
export type Slot = 0 | 1 | 2;
export type Mode = 'full' | 'short';

export type Cell = { kind: 'count'; n: number; pct: string } | { kind: 'mean'; mean: number; sd: number };
export interface FactRow { label: string; group?: string; drug: Cell; placebo: Cell; total: Cell }
export interface FactPassage { id: PassageId; topic: string; armN: { drug: number; placebo: number; total: number }; rows: FactRow[] }
export interface FactPack { study: string; drug: string; indication: string; passages: FactPassage[] }

export interface VariantChecks { numbersOk: boolean; lengthOk: boolean; structureOk: boolean; orderingOk: boolean; humanOk: boolean | null }
export interface Variant { id: string; passageId: TrainingPassageId; axes: AxisCombo; text: string; wordCount: number; tries: number; checks: VariantChecks; note: string }
export type Bank = Variant[];

export interface AxisConfig { name: string; poles: [string, string]; rules: [{ firm: string; soft: string }, { firm: string; soft: string }]; phrases: [string, string] }
export type AxesConfig = Record<Axis, AxisConfig>;

export interface ScheduledRound { n: number; passageId: TrainingPassageId; tested: [Axis, Axis]; repeatOf: number | null }
export interface ShownCard { variantId: string; slot: Slot }
export const REASON_CHIPS = ['More concise', 'Better flow', 'Right level of detail', 'Too much data', 'Wrong order', 'Other'] as const;
export type ReasonChip = (typeof REASON_CHIPS)[number];
export interface Round extends ScheduledRound { shown: ShownCard[]; pick: string | 'none' | null; reasonChips: ReasonChip[]; latencyMs: number | null; shownAt: string }

export type RuleStrength = 'firm' | 'soft';
export interface Rule { axis: Axis; pole: Pole; strength: RuleStrength }
export interface Profile { scores: Record<Axis, number>; timesTested: Record<Axis, number>; rules: Rule[]; styleBlock: string; summary: string; consistent: boolean | null }

export type DraftLabel = 'A' | 'B' | 'C';
export interface Draft { label: DraftLabel; slot: Slot; text: string; system: string; tries: number }
export interface EvalTrial { passageId: EvalPassageId; drafts: Draft[]; best: DraftLabel | null; worst: DraftLabel | null; cSource: string; bEqualsA: boolean }
export type EvalStatus = 'not-started' | 'pending' | 'ready' | 'failed';
export interface Workload { mental: number; physical: number; temporal: number; performance: number; effort: number; frustration: number }

export interface Session {
  id: string; participantId: string; mode: Mode; retestOf: string | null;
  startedAt: string; finishedAt: string | null;
  rounds: Round[]; profile: Profile | null;
  evalStatus: EvalStatus; evals: EvalTrial[];
  workload: Workload | null; ruleFeedback: Partial<Record<Axis, 'up' | 'down'>>;
}
```

- Produces, in `src/core/factPack.ts`:
  - `validateFactPack(pack: FactPack): string[]` returns problem messages, or `[]` if the pack is valid
  - `loadFactPack(path: string): FactPack` throws `Error` listing every problem
  - `allowedNumbers(passage: FactPassage): Set<string>`
  - `formatPct(n: number, of: number): string`

- [ ] **Step 1: Scaffold the project**

`package.json` scripts: `test` = `vitest run`, `typecheck` = `tsc --noEmit`, `e2e` = `playwright test`, `build` = `vite build`, `start` = `tsx src/server/index.ts`, `dev:server` = `tsx watch src/server/index.ts`, `dev:client` = `vite`, `bank` = `tsx src/scripts/bank-cli.ts generate`, `bank:export-review` = `tsx src/scripts/bank-cli.ts export-review`, `bank:import-review` = `tsx src/scripts/bank-cli.ts import-review`, `bank:check` = `tsx src/scripts/bank-cli.ts check`, `llm:smoke` = `tsx src/scripts/llm-smoke.ts`, `persona` = `tsx src/scripts/persona-run.ts`. Install the dependencies named in Tech Stack plus `@types/express`, `@types/supertest` and `@types/node`. `tsconfig.json` needs `"strict": true`, `"module": "NodeNext"` and `"target": "ES2022"`. `.gitignore` covers `node_modules`, `dist`, `data`, `.e2e-data`. Write `src/core/types.ts` exactly as above.

Run: `npm run typecheck`
Expected: exits 0

- [ ] **Step 2: Write the failing tests**

The fixture `tests/fixtures/pack.ts` exports `fixturePack(): FactPack`. It holds P1 with exactly the rows below, and P2, P3, E1, E2 with one valid count row each.

| Row (group) | Drug (n=206) | Placebo (n=206) | Total (n=412) |
|---|---|---|---|
| Age, years | mean 54.1, SD 11.0 | mean 54.6, SD 11.4 | mean 54.4, SD 11.2 |
| <65 years (age) | 160, 77.7% | 158, 76.7% | 318, 77.2% |
| ≥65 years (age) | 46, 22.3% | 48, 23.3% | 94, 22.8% |
| Female (sex) | 119, 57.8% | 120, 58.3% | 239, 58.0% |
| Male (sex) | 87, 42.2% | 86, 41.7% | 173, 42.0% |

```ts
it('accepts the fixture pack', () => expect(validateFactPack(fixturePack())).toEqual([]));
it('accepts the shipped fact pack', () => expect(() => loadFactPack('content/fact-pack.json')).not.toThrow());
it('flags a wrong percentage', () => {
  const p = fixturePack(); p.passages[0].rows[3].drug = { kind: 'count', n: 119, pct: '57.9%' };
  expect(validateFactPack(p)).toContain('P1 / Female / drug: pct 57.9% should be 57.8%');
});
it('flags a total that is not drug + placebo', () => {
  const p = fixturePack(); p.passages[0].rows[3].total = { kind: 'count', n: 240, pct: '58.3%' };
  expect(validateFactPack(p)).toContain('P1 / Female / total: n 240 should be 239');
});
it('flags a group that does not add up to the arm size', () => {
  const p = fixturePack(); p.passages[0].rows[4].placebo = { kind: 'count', n: 85, pct: '41.3%' };
  expect(validateFactPack(p)).toContain('P1 / group sex / placebo: 205 should be 206');
});
it('flags a total mean more than 0.1 from the weighted mean', () => {
  const p = fixturePack(); p.passages[0].rows[0].total = { kind: 'mean', mean: 55.0, sd: 11.2 };
  expect(validateFactPack(p)).toContain('P1 / Age, years / total: mean 55 should be 54.35 ± 0.1');
});
it('flags a missing passage', () => {
  const p = fixturePack(); p.passages = p.passages.filter(x => x.id !== 'E2');
  expect(validateFactPack(p)).toContain('missing passage E2');
});
it('collects every number a text may use', () => {
  const a = allowedNumbers(fixturePack().passages[0]);
  for (const x of ['206', '412', '119', '57.8', '54.1', '11.0', '65']) expect(a.has(x)).toBe(true);
  expect(a.has('57.8%')).toBe(false); // stored without %, checks strip it
});
it('formats percentages to one decimal', () => expect(formatPct(86, 206)).toBe('41.7%'));
```

- [ ] **Step 3: Run tests to verify they fail**

Run: `npx vitest run tests/core/factPack.test.ts`
Expected: FAIL, cannot find module `../../src/core/factPack`

- [ ] **Step 4: Implement `src/core/factPack.ts` and write `content/fact-pack.json`**

`formatPct(n, of)` = `(Math.round((n / of) * 1000) / 10).toFixed(1) + '%'`. A weighted mean is `(drug.mean * armN.drug + placebo.mean * armN.placebo) / armN.total`, and messages print it with `toFixed(2)`. `allowedNumbers` collects every count, every pct with `%` stripped, every mean and SD as written with `toFixed(1)`, the three arm sizes, and every number found in row labels. Write `content/fact-pack.json` with the study frame from Global Constraints and these passages and rows (you pick the numbers, and the validator must pass):

| ID | Topic | Rows (group) |
|---|---|---|
| P1 | Age and sex | Age, years (mean); <65 years, ≥65 years (age); Female, Male (sex) |
| P2 | Race and ethnicity | White, Black or African American, Asian, Other or multiple (race); Hispanic or Latino |
| P3 | Baseline disease characteristics | Disease duration, years (mean); PASI score (mean); Body surface area affected, % (mean); IGA 3, IGA 4 (iga) |
| E1 | Weight and BMI | Weight, kg (mean); BMI, kg/m2 (mean); BMI ≥30 |
| E2 | Prior therapy | Prior systemic therapy; Prior biologic therapy; Prior phototherapy |

- [ ] **Step 5: Run tests to verify they pass**

Run: `npx vitest run tests/core/factPack.test.ts && npm run typecheck`
Expected: PASS, then exit 0

- [ ] **Step 6: Commit**

```bash
git add poc
git commit -m "feat(poc): scaffold, shared types, synthetic fact pack"
```

---

### Task 2: Text checks

**Files:**
- Create: `poc/src/core/checks.ts`
- Test: `poc/tests/core/checks.test.ts`

**Interfaces:**
- Consumes: `Pole` from Task 1.
- Produces:
  - `extractNumbers(text: string): string[]`
  - `checkNumbers(text: string, allowed: Set<string>): { ok: boolean; unknown: string[] }`
  - `wordCount(text: string): number`
  - `checkLength(text: string, density: Pole): boolean`
  - `hasListOrTable(text: string): boolean`
  - `checkStructure(text: string, structure: Pole): boolean`
  - `spreadOutliers(items: Array<{ id: string; wordCount: number }>): string[]`

- [ ] **Step 1: Write the failing tests**

```ts
const allowed = new Set(['412', '57.8', '119']);
it('extracts numbers with their percent signs', () =>
  expect(extractNumbers('Of 412 patients, 57.8% were female (n=119).')).toEqual(['412', '57.8%', '119']));
it('passes text whose numbers are all allowed', () =>
  expect(checkNumbers('Of 412 patients, 57.8% were female.', allowed)).toEqual({ ok: true, unknown: [] }));
it('reports numbers not in the fact pack', () =>
  expect(checkNumbers('Of 413 patients, 57.8% were female.', allowed)).toEqual({ ok: false, unknown: ['413'] }));
it('counts words, ignoring bullets and table bars', () =>
  expect(wordCount('- Female: 119 (57.8%)\n| Age | 54.1 |')).toBe(5));
it('applies the density word bands', () => {
  const w = (n: number) => Array(n).fill('word').join(' ');
  expect([checkLength(w(49), 0), checkLength(w(50), 0), checkLength(w(90), 0), checkLength(w(91), 0)]).toEqual([false, true, true, false]);
  expect([checkLength(w(109), 1), checkLength(w(110), 1), checkLength(w(160), 1), checkLength(w(161), 1)]).toEqual([false, true, true, false]);
});
it('detects bullets, numbered lists and tables', () => {
  expect(['- a', '* a', '• a', '1. a', '| a | b |'].map(hasListOrTable)).toEqual([true, true, true, true, true]);
  expect(hasListOrTable('Prose only. A 54.1-year mean age.')).toBe(false);
});
it('matches structure to its pole', () => {
  expect(checkStructure('Prose only.', 0)).toBe(true);
  expect(checkStructure('- Female: 119', 0)).toBe(false);
  expect(checkStructure('- Female: 119', 1)).toBe(true);
});
it('flags texts outside ±15% of the group mean', () =>
  expect(spreadOutliers([{ id: 'a', wordCount: 100 }, { id: 'b', wordCount: 100 }, { id: 'c', wordCount: 130 }])).toEqual(['c']));
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx vitest run tests/core/checks.test.ts`
Expected: FAIL, cannot find module

- [ ] **Step 3: Implement `src/core/checks.ts`**

Numbers match `/\d+(?:\.\d+)?%?/g`. `checkNumbers` strips a trailing `%` before the lookup and reports `unknown` in the original form. `wordCount` splits on whitespace and `|` and counts tokens that contain a letter or digit. A list or table line starts (after optional spaces) with `- `, `* `, `• `, or `\d+\. `, or contains two or more `|`. For `spreadOutliers`, an outlier is above `1.15 × mean` or below `0.85 × mean`.

- [ ] **Step 4: Run tests to verify they pass**

Run: `npx vitest run tests/core/checks.test.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add poc/src/core/checks.ts poc/tests/core/checks.test.ts
git commit -m "feat(poc): number, length and structure checks"
```

---

### Task 3: Round schedule and card picking

**Files:**
- Create: `poc/src/core/rng.ts`, `poc/src/core/schedule.ts`
- Test: `poc/tests/core/schedule.test.ts`

**Interfaces:**
- Consumes: types from Task 1.
- Produces:
  - `type Rng = () => number`, `seededRng(seed: number): Rng`, `shuffle<T>(items: readonly T[], rng: Rng): T[]` (in `rng.ts`)
  - `scheduleFor(mode: Mode): ScheduledRound[]`
  - `scoredRoundCount(mode: Mode): number` (6 for full, 4 for short)
  - `variantId(passageId: TrainingPassageId, combo: AxisCombo): string`
  - `parseVariantId(id: string): { passageId: TrainingPassageId; combo: AxisCombo }`
  - `pickCards(round: ScheduledRound, lean: Record<Axis, Pole | null>, rng: Rng): ShownCard[]` (sorted by slot)
  - `repeatCards(original: ShownCard[], rng: Rng): ShownCard[]` (sorted by slot)

- [ ] **Step 1: Write the failing tests**

```ts
const row = (r: ScheduledRound) => [r.n, r.passageId, r.tested.join(''), r.repeatOf];
it('full schedule covers every axis pair once, then repeats round 2', () =>
  expect(scheduleFor('full').map(row)).toEqual([
    [1, 'P1', 'DS', null], [2, 'P2', 'OH', null], [3, 'P3', 'DO', null],
    [4, 'P1', 'SH', null], [5, 'P2', 'DH', null], [6, 'P3', 'SO', null], [7, 'P2', 'OH', 2]]));
it('short schedule is rounds 1–4 then the repeat', () =>
  expect(scheduleFor('short').map(row)).toEqual([
    [1, 'P1', 'DS', null], [2, 'P2', 'OH', null], [3, 'P3', 'DO', null], [4, 'P1', 'SH', null], [5, 'P2', 'OH', 2]]));
it('round-trips variant IDs', () => {
  const combo = { D: 0, S: 1, O: 0, H: 1 } as const;
  expect(variantId('P1', combo)).toBe('P1-D0S1O0H1');
  expect(parseVariantId('P1-D0S1O0H1')).toEqual({ passageId: 'P1', combo });
});
it('picks three valid cards for any seed', () => {
  const r = scheduleFor('full')[0];
  const noLean = { D: null, S: null, O: null, H: null };
  for (let seed = 1; seed <= 50; seed++) {
    const cards = pickCards(r, noLean, seededRng(seed));
    const combos = cards.map(c => parseVariantId(c.variantId));
    expect(cards.map(c => c.slot)).toEqual([0, 1, 2]);
    expect(new Set(cards.map(c => c.variantId)).size).toBe(3);
    expect(combos.every(c => c.passageId === 'P1')).toBe(true);
    for (const a of ['O', 'H'] as const) expect(new Set(combos.map(c => c.combo[a])).size).toBe(1); // untested axes held
    for (const a of ['D', 'S'] as const) expect(new Set(combos.map(c => c.combo[a])).size).toBe(2); // tested axes show both poles
  }
});
it('holds untested axes at the current lean', () => {
  const cards = pickCards(scheduleFor('full')[0], { D: null, S: null, O: 1, H: 0 }, seededRng(7));
  expect(cards.every(c => parseVariantId(c.variantId).combo.O === 1 && parseVariantId(c.variantId).combo.H === 0)).toBe(true);
});
it('repeat shows the same cards in a different order', () => {
  for (let seed = 1; seed <= 50; seed++) {
    const original = pickCards(scheduleFor('full')[1], { D: 0, S: 0, O: null, H: null }, seededRng(seed));
    const again = repeatCards(original, seededRng(seed + 100));
    expect(new Set(again.map(c => c.variantId))).toEqual(new Set(original.map(c => c.variantId)));
    expect(again.some((c, i) => c.variantId !== original[i].variantId)).toBe(true);
  }
});
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx vitest run tests/core/schedule.test.ts`
Expected: FAIL, cannot find module

- [ ] **Step 3: Implement `src/core/rng.ts` and `src/core/schedule.ts`**

Use mulberry32 for `seededRng` and Fisher–Yates for `shuffle`. `pickCards` picks one pole for each untested axis once per round (the lean, or `rng() < 0.5` when the lean is null). It builds the 4 tested combinations, drops one at random, and shuffles the other 3 into slots 0–2. `repeatCards` reshuffles until the order differs from the original.

- [ ] **Step 4: Run tests to verify they pass**

Run: `npx vitest run tests/core/schedule.test.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add poc/src/core/rng.ts poc/src/core/schedule.ts poc/tests/core/schedule.test.ts
git commit -m "feat(poc): fixed round schedule and card picking"
```

---

### Task 4: Scoring and rules

**Files:**
- Create: `poc/src/core/scoring.ts`
- Test: `poc/tests/core/scoring.test.ts`

**Interfaces:**
- Consumes: `Round`, `Rule`, `Axis`, `Pole` (Task 1), `parseVariantId` (Task 3).
- Produces:
  - `type Votes = Record<Axis, [number, number]>` (votes for pole 0, votes for pole 1)
  - `scoreRounds(rounds: Round[]): { votes: Votes; timesTested: Record<Axis, number>; scores: Record<Axis, number> }`
  - `decideRules(votes: Votes, timesTested: Record<Axis, number>): Rule[]` (in `AXES` order)
  - `currentLean(votes: Votes): Record<Axis, Pole | null>`
  - `isConsistent(rounds: Round[]): boolean | null`

Rules: only rounds with `repeatOf === null` and a non-null `pick` count. `timesTested` counts every such round that tested the axis, `none` included. `score = (votes[1] − votes[0]) / timesTested` (0 when untested). `lead = max(votes)`. A rule exists when `lead ≥ 2` and `lead > min(votes)`. It's `firm` when `lead === timesTested` and `soft` otherwise.

- [ ] **Step 1: Write the failing tests**

The helper `r(n, tested, pick, repeatOf = null)` builds a `Round` on passage P1 whose `pick` is `variantId('P1', pick)`, or `'none'`.

```ts
const A = { D: 0, S: 0, O: 0, H: 0 } as const;
const fullAllA = scheduleFor('full').slice(0, 6).map(s => r(s.n, s.tested, A));
it('all pole-0 picks give -1 everywhere and four firm pole-0 rules', () => {
  const s = scoreRounds(fullAllA);
  expect(s.scores).toEqual({ D: -1, S: -1, O: -1, H: -1 });
  expect(s.timesTested).toEqual({ D: 3, S: 3, O: 3, H: 3 });
  expect(decideRules(s.votes, s.timesTested)).toEqual(AXES.map(axis => ({ axis, pole: 0, strength: 'firm' })));
});
it('2 of 3 is a soft rule', () => {
  const votes = { D: [1, 2], S: [0, 0], O: [0, 0], H: [0, 0] } as Votes;
  expect(decideRules(votes, { D: 3, S: 3, O: 3, H: 3 })).toEqual([{ axis: 'D', pole: 1, strength: 'soft' }]);
});
it('1 vote and 2 nones is no rule', () =>
  expect(decideRules({ D: [0, 1], S: [0, 0], O: [0, 0], H: [0, 0] }, { D: 3, S: 3, O: 3, H: 3 })).toEqual([]));
it('short mode: 2 of 2 is firm, 1 of 2 is nothing', () => {
  expect(decideRules({ D: [2, 0], S: [1, 1], O: [0, 0], H: [0, 0] }, { D: 2, S: 2, O: 2, H: 2 }))
    .toEqual([{ axis: 'D', pole: 0, strength: 'firm' }]);
});
it('all none gives zero scores and no rules', () => {
  const s = scoreRounds(scheduleFor('full').slice(0, 6).map(x => r(x.n, x.tested, 'none')));
  expect(s.scores).toEqual({ D: 0, S: 0, O: 0, H: 0 });
  expect(decideRules(s.votes, s.timesTested)).toEqual([]);
});
it('the repeat round never votes', () => {
  const withRepeat = [...fullAllA, r(7, ['O', 'H'], { D: 1, S: 1, O: 1, H: 1 }, 2)];
  expect(scoreRounds(withRepeat).scores).toEqual(scoreRounds(fullAllA).scores);
});
it('consistency compares the repeat with its original', () => {
  const base = fullAllA;
  expect(isConsistent(base)).toBeNull();
  expect(isConsistent([...base, r(7, ['O', 'H'], A, 2)])).toBe(true);
  expect(isConsistent([...base, r(7, ['O', 'H'], { ...A, O: 1 }, 2)])).toBe(false);
  const noneBase = base.map(x => (x.n === 2 ? { ...x, pick: 'none' } : x));
  expect(isConsistent([...noneBase, r(7, ['O', 'H'], 'none', 2)])).toBe(true);
});
it('lean follows the majority, null on ties', () =>
  expect(currentLean({ D: [2, 1], S: [1, 1], O: [0, 0], H: [0, 3] })).toEqual({ D: 0, S: null, O: null, H: 1 }));
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx vitest run tests/core/scoring.test.ts`
Expected: FAIL, cannot find module

- [ ] **Step 3: Implement `src/core/scoring.ts`** to the rules above.

- [ ] **Step 4: Run tests to verify they pass**

Run: `npx vitest run tests/core/scoring.test.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add poc/src/core/scoring.ts poc/tests/core/scoring.test.ts
git commit -m "feat(poc): vote counting, rules and consistency"
```

---

### Task 5: Axes content, style block and profile

**Files:**
- Create: `poc/content/axes.json`, `poc/src/core/profile.ts`
- Test: `poc/tests/core/profile.test.ts`

**Interfaces:**
- Consumes: Task 1 types, `scoreRounds`, `decideRules`, `isConsistent` (Task 4).
- Produces:
  - `loadAxesConfig(path: string): AxesConfig`
  - `ruleText(rule: Rule, axes: AxesConfig): string`
  - `styleBlock(rules: Rule[], axes: AxesConfig): string`
  - `profileSummary(rules: Rule[], axes: AxesConfig): string`
  - `buildProfile(rounds: Round[], axes: AxesConfig): Profile`
  - `mirrorProfile(p: Profile, axes: AxesConfig): Profile`

- [ ] **Step 1: Write `content/axes.json` with exactly this content**

| Axis | Pole | `poles[i]` (goes into the bank prompt) | `rules[i].firm` | `rules[i].soft` | `phrases[i]` |
|---|---|---|---|---|---|
| D Density | 0 | Terse. State facts only. No intro sentence, no transitions, no restating what a table would show. 50–90 words. | Write tersely. State the facts with no introduction, transitions, or restated table content. | Lean toward terse writing with little framing. | terse |
| D | 1 | Explanatory. Full sentences that say what the numbers mean, with brief context. 110–160 words. | Explain what the numbers mean in full sentences, with brief context. | Lean toward explaining what the numbers mean. | explanatory |
| S Structure | 0 | Prose paragraphs only. No bullets, no tables. | Use prose paragraphs. Do not use bullets or tables. | Prefer prose over bullets or tables. | prose |
| S | 1 | Bullets or a compact table for the numbers, with one lead-in sentence. | Present the numbers as bullets or a compact table, with one lead-in sentence. | Prefer bullets or a compact table for the numbers. | bullets or tables |
| O Ordering | 0 | Lead with the main takeaway in the first sentence (for example, that the arms were balanced). Details after. | Put the main takeaway in the first sentence, then the details. | Lean toward stating the main takeaway early. | main point first |
| O | 1 | Walk through the characteristics first. State the main takeaway in the last sentence. | Walk through the details first and state the main takeaway in the last sentence. | Lean toward building up to the main takeaway. | main point last |
| H Data handling | 0 | Selective. Report only the 3–4 numbers that matter most and say what they mean. | Report only the three or four most important numbers and say what they mean. | Lean toward fewer numbers, each interpreted. | only the key numbers |
| H | 1 | Exhaustive. Report every number given for this topic. | Report every number given for the topic. | Lean toward reporting all the numbers given. | every number |

- [ ] **Step 2: Write the failing tests**

```ts
const axes = loadAxesConfig('content/axes.json');
const D0f = { axis: 'D', pole: 0, strength: 'firm' } as const;
const H1s = { axis: 'H', pole: 1, strength: 'soft' } as const;
it('looks up rule text', () =>
  expect(ruleText(D0f, axes)).toBe('Write tersely. State the facts with no introduction, transitions, or restated table content.'));
it('empty style block when there are no rules', () => expect(styleBlock([], axes)).toBe(''));
it('builds the style block', () =>
  expect(styleBlock([D0f, H1s], axes)).toBe(
    'This writer has these style preferences. Follow them unless they conflict with the facts.\n' +
    '- Write tersely. State the facts with no introduction, transitions, or restated table content.\n' +
    '- Lean toward reporting all the numbers given.\n' +
    'Do not change facts, numbers, or terminology from the source material.'));
it('summarises the profile in plain words', () => {
  expect(profileSummary([], axes)).toBe("Your picks didn't show a clear style yet.");
  expect(profileSummary([D0f, { axis: 'S', pole: 0, strength: 'firm' }, { axis: 'O', pole: 0, strength: 'soft' }], axes))
    .toBe('Your style: terse, prose, main point first (leaning)');
});
it('mirror flips every pole and score and rebuilds the text', () => {
  const p: Profile = { scores: { D: -1, S: 0, O: 0, H: 1 / 3 }, timesTested: { D: 3, S: 3, O: 3, H: 3 }, rules: [D0f, H1s],
    styleBlock: styleBlock([D0f, H1s], axes), summary: profileSummary([D0f, H1s], axes), consistent: true };
  const m = mirrorProfile(p, axes);
  expect(m.rules).toEqual([{ axis: 'D', pole: 1, strength: 'firm' }, { axis: 'H', pole: 0, strength: 'soft' }]);
  expect(m.scores.D).toBe(1);
  expect(m.styleBlock).toBe(styleBlock(m.rules, axes));
});
```

- [ ] **Step 3: Run tests to verify they fail**

Run: `npx vitest run tests/core/profile.test.ts`
Expected: FAIL, cannot find module

- [ ] **Step 4: Implement `src/core/profile.ts`**

The summary joins phrases in `AXES` order with `, `, adds ` (leaning)` after soft rules, and prefixes `Your style: `. `buildProfile` composes Task 4 and these helpers.

- [ ] **Step 5: Run tests to verify they pass**

Run: `npx vitest run tests/core/profile.test.ts`
Expected: PASS

- [ ] **Step 6: Commit**

```bash
git add poc/content/axes.json poc/src/core/profile.ts poc/tests/core/profile.test.ts
git commit -m "feat(poc): axes content, style block and profile"
```

---

### Task 6: LLM client and prompts

**Files:**
- Create: `poc/src/llm/client.ts`, `poc/src/llm/prompts.ts`, `poc/src/scripts/llm-smoke.ts`
- Test: `poc/tests/llm/prompts.test.ts`, `poc/tests/llm/client.test.ts`

**Interfaces:**
- Consumes: `FactPassage`, `AxisCombo`, `AxesConfig` (Task 1).
- Produces, in `client.ts`:
  - `interface LlmRequest { system: string; user: string }`
  - `interface LlmClient { complete(req: LlmRequest): Promise<string> }`
  - `anthropicClient(opts: { apiKey: string; model: string; temperature: number; maxTokens: number }): LlmClient`
  - `fakeClient(respond: (req: LlmRequest) => string | Error): LlmClient & { calls: LlmRequest[] }`
  - `clientFromEnv(env: NodeJS.ProcessEnv): LlmClient`. When `LLM_FAKE=1`, it's a fake. It answers `YES` when `system === ORDERING_SYSTEM`, `1` when the system prompt starts with `You are a medical writer who` (the persona script, Task 13), and `The study enrolled 412 patients.` to everything else. Otherwise it's `anthropicClient` with `ANTHROPIC_API_KEY`, `LLM_MODEL`, `temperature: 0.2`, `maxTokens: 600`.
- Produces, in `prompts.ts`:
  - `BANK_SYSTEM: string`, `ORDERING_SYSTEM: string`, `BANNED_PROMPT_WORDS: readonly string[]`
  - `bankUserPrompt(passage: FactPassage, combo: AxisCombo, axes: AxesConfig): string`
  - `orderingCheck(text: string): LlmRequest`
  - `parseYesNo(reply: string): boolean | null`
  - `evalSystemPrompt(baseline: string, styleBlock: string): string`
  - `evalUserPrompt(passage: FactPassage): string`

- [ ] **Step 1: Write the failing tests**

`BANK_SYSTEM` is exactly:
```
You are a medical writer drafting one part of the "Demographic and Other Baseline Characteristics" section (ICH E3 section 11.2) of a clinical study report. Use only the facts given. Do not add, drop, or re-round numbers except as the style asks. Do not change any terms.
```
`ORDERING_SYSTEM` is exactly: `Answer with YES or NO only.` The ordering user prompt is `Is the main takeaway of this paragraph stated in its first sentence?\n\n{text}`.

```ts
it('B is A plus the style block, nothing else', () => {
  expect(evalSystemPrompt('BASE', '')).toBe('BASE');
  expect(evalSystemPrompt('BASE', 'BLOCK')).toBe('BASE\n\nBLOCK');
});
it('bank prompt carries the facts and all four pole texts', () => {
  const combo = { D: 0, S: 1, O: 0, H: 1 } as const;
  const u = bankUserPrompt(p1, combo, axes);
  expect(u).toContain(JSON.stringify(p1.rows));
  for (const a of AXES) expect(u).toContain(axes[a].poles[combo[a]]);
});
it('no prompt we write leaks the experiment', () => {
  const texts = [BANK_SYSTEM, ORDERING_SYSTEM, evalUserPrompt(e1), orderingCheck('x').user,
    ...allCombos().map(c => bankUserPrompt(p1, c, axes))];
  for (const t of texts) for (const w of BANNED_PROMPT_WORDS) expect(t.toLowerCase()).not.toContain(w);
});
it('parses yes/no', () => expect(['YES', 'No.', 'maybe'].map(parseYesNo)).toEqual([true, false, null]));
it('fake client records calls and can fail', async () => {
  const f = fakeClient(req => (req.user === 'boom' ? new Error('down') : 'ok'));
  await expect(f.complete({ system: 's', user: 'hi' })).resolves.toBe('ok');
  await expect(f.complete({ system: 's', user: 'boom' })).rejects.toThrow('down');
  expect(f.calls.length).toBe(2);
});
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx vitest run tests/llm`
Expected: FAIL, cannot find module

- [ ] **Step 3: Implement `client.ts` and `prompts.ts`**

`bankUserPrompt` returns `Facts:\n{JSON.stringify(passage.rows)}\n\nWrite this part of the section in this style:\n- {D pole}\n- {S pole}\n- {O pole}\n- {H pole}`. `evalUserPrompt` returns `Facts:\n{JSON.stringify(passage.rows)}\n\nDraft this part of the demographics section.` `anthropicClient` calls `messages.create` with `system`, one user message, `temperature` and `max_tokens`, and joins the text blocks of the reply. `llm-smoke.ts` sends `evalUserPrompt` for E1 through `clientFromEnv(process.env)` and prints the reply.

- [ ] **Step 4: Run tests to verify they pass**

Run: `npx vitest run tests/llm && npm run typecheck`
Expected: PASS, then exit 0

- [ ] **Step 5: Commit**

```bash
git add poc/src/llm poc/src/scripts/llm-smoke.ts poc/tests/llm
git commit -m "feat(poc): LLM client and prompts"
```

---

### Task 7: Paragraph bank: generate, review, readiness

**Files:**
- Create: `poc/src/bank/generate.ts`, `poc/src/bank/review.ts`, `poc/src/scripts/bank-cli.ts`
- Create: `poc/tests/fixtures/bank.ts`
- Test: `poc/tests/bank/generate.test.ts`, `poc/tests/bank/review.test.ts`

**Interfaces:**
- Consumes: Tasks 1, 2, 3 (`variantId`), 5 (`AxesConfig`), 6 (`LlmClient`, prompts).
- Produces:
  - `class BankError extends Error { id: string }`
  - `generateVariant(passage: FactPassage, combo: AxisCombo, axes: AxesConfig, llm: LlmClient, maxTries: number): Promise<Variant>` (`humanOk: null`)
  - `generateBank(pack: FactPack, axes: AxesConfig, llm: LlmClient, opts: { maxTries: number; only?: string[]; existing?: Bank }): Promise<{ bank: Bank; failed: string[] }>`
  - `bankReady(bank: Bank): { ready: boolean; problems: string[] }`
  - `exportReviewCsv(bank: Bank): string`, `applyReviewCsv(bank: Bank, csv: string): Bank`
  - `fixtureBank(wordsPerText?: number): Bank` (in `tests/fixtures/bank.ts`): 48 variants, every check `true`, each text `wordsPerText` words (default 70) that include `412`
- CLI (`bank-cli.ts`): `generate [--only id,id]` writes `content/bank.json` and prints failed IDs. `export-review` writes `content/bank-review.csv`. `import-review` applies it. `check` prints `Bank ready: 48 texts`, or the problems, and exits 1.

- [ ] **Step 1: Write the failing tests**

```ts
const ordering = (yes: boolean) => (req: LlmRequest) => (req.system === ORDERING_SYSTEM ? (yes ? 'YES' : 'NO') : undefined);
it('retries until a reply passes every check', async () => {
  let n = 0;
  const llm = fakeClient(req => ordering(true)(req) ?? (++n === 1 ? words(70, '999') : words(70, '412')));
  const v = await generateVariant(p1, { D: 0, S: 0, O: 0, H: 0 }, axes, llm, 4);
  expect(v.id).toBe('P1-D0S0O0H0');
  expect(v.tries).toBe(2);
  expect(v.checks).toEqual({ numbersOk: true, lengthOk: true, structureOk: true, orderingOk: true, humanOk: null });
});
it('checks ordering against the pole', async () => {
  const llm = fakeClient(req => ordering(true)(req) ?? words(70, '412'));
  await expect(generateVariant(p1, { D: 0, S: 0, O: 1, H: 0 }, axes, llm, 2)).rejects.toThrow(BankError);
});
it('regenerates only the listed IDs', async () => {
  const existing = fixtureBank();
  const llm = fakeClient(req => ordering(true)(req) ?? words(70, '412'));
  const { bank } = await generateBank(fixturePack(), axes, llm, { maxTries: 2, only: ['P1-D0S0O0H0'], existing });
  expect(bank.filter((v, i) => v !== existing[i]).map(v => v.id)).toEqual(['P1-D0S0O0H0']);
});
it('bank is ready only with 48 fully checked, human-approved texts', () => {
  expect(bankReady(fixtureBank())).toEqual({ ready: true, problems: [] });
  const b = fixtureBank(); b[0] = { ...b[0], checks: { ...b[0].checks, humanOk: null } };
  expect(bankReady(b).problems).toContain('P1-D0S0O0H0: not reviewed');
  expect(bankReady(fixtureBank().slice(1)).problems).toContain('missing P1-D0S0O0H0');
});
it('review CSV round-trips and records rejections', () => {
  const b = fixtureBank();
  const csv = exportReviewCsv(b);
  expect(csv.split('\n')[0]).toBe('id,humanOk,note,text');
  const edited = csv.replace('P1-D0S0O0H0,,', 'P1-D0S0O0H0,no,sounds robotic');
  const after = applyReviewCsv(b, edited);
  expect(after[0].checks.humanOk).toBe(false);
  expect(after[0].note).toBe('sounds robotic');
});
```
(`words(n, number)` is a test helper that returns `n` words including `number`.)

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx vitest run tests/bank`
Expected: FAIL, cannot find module

- [ ] **Step 3: Implement `generate.ts`, `review.ts` and `bank-cli.ts`**

A reply passes when `checkNumbers` against `allowedNumbers(passage)`, `checkLength` for D, `checkStructure` for S, and the ordering check (YES for O=0, NO for O=1) all pass. `bankReady` also runs `spreadOutliers` within each passage and Density pole group (8 texts each) and reports `"{id}: length outside ±15% of its group"`. In the CSV, `humanOk` is blank for unreviewed, `yes` or `no` otherwise.

- [ ] **Step 4: Run tests to verify they pass**

Run: `npx vitest run tests/bank`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add poc/src/bank poc/src/scripts/bank-cli.ts poc/tests/bank poc/tests/fixtures/bank.ts
git commit -m "feat(poc): paragraph bank generation, review and readiness"
```

---

### Task 8: Eval drafts

**Files:**
- Create: `poc/content/baseline-prompt.txt` (contents of `BANK_SYSTEM` until Q12 is answered)
- Create: `poc/src/server/evalDrafts.ts`
- Test: `poc/tests/server/evalDrafts.test.ts`

**Interfaces:**
- Consumes: Tasks 1, 2, 3 (`shuffle`, `Rng`), 5 (`mirrorProfile`), 6 (prompts, `LlmClient`).
- Produces:
  - `class DraftError extends Error {}`
  - `chooseCProfile(self: Session, others: Session[], axes: AxesConfig): { profile: Profile; source: string }`
  - `buildTrial(passage: FactPassage, baseline: string, self: Profile, c: { profile: Profile; source: string }, llm: LlmClient, rng: Rng, maxTries: number): Promise<EvalTrial>`
  - `prepareEvals(session: Session, deps: { pack: FactPack; baseline: string; axes: AxesConfig; llm: LlmClient; rng: Rng; others: Session[] }): Promise<Session>` returns the session with `evalStatus` `'ready'` and two trials (E1, E2), or `'failed'` and `evals: []`

- [ ] **Step 1: Write the failing tests**

```ts
it('chooseCProfile falls back to a mirror with nobody else', () =>
  expect(chooseCProfile(me, [], axes).source).toBe('mirror'));
it('chooseCProfile takes the newest other profile that differs', () => {
  const older = sessionWith('W01', '2026-10-27T10:00:00Z', otherProfile);
  const newer = sessionWith('W02', '2026-10-27T11:00:00Z', otherProfile2);
  const same = sessionWith('W03', '2026-10-27T12:00:00Z', me.profile!);
  expect(chooseCProfile(me, [older, newer, same], axes).source).toBe('W02');
});
it('builds A, B and C with one shared user prompt', async () => {
  const llm = fakeClient(() => 'The study enrolled 412 patients.');
  const t = await buildTrial(e1, 'BASE', me.profile!, { profile: otherProfile, source: 'W01' }, llm, seededRng(1), 3);
  expect(t.drafts.map(d => d.label).sort()).toEqual(['A', 'B', 'C']);
  expect(t.drafts.map(d => d.slot).sort()).toEqual([0, 1, 2]);
  const by = Object.fromEntries(t.drafts.map(d => [d.label, d.system]));
  expect(by.A).toBe('BASE');
  expect(by.B).toBe(evalSystemPrompt('BASE', me.profile!.styleBlock));
  expect(by.C).toBe(evalSystemPrompt('BASE', otherProfile.styleBlock));
  expect(new Set(llm.calls.map(c => c.user)).size).toBe(1);
  expect(t.bEqualsA).toBe(false);
});
it('flags bEqualsA when the person has no rules', async () => {
  const t = await buildTrial(e1, 'BASE', noRulesProfile, { profile: otherProfile, source: 'W01' }, fakeClient(() => 'The study enrolled 412 patients.'), seededRng(1), 3);
  expect(t.bEqualsA).toBe(true);
});
it('retries a draft with an unknown number, then gives up', async () => {
  await expect(buildTrial(e1, 'BASE', me.profile!, { profile: otherProfile, source: 'W01' }, fakeClient(() => '999 patients.'), seededRng(1), 3))
    .rejects.toThrow(DraftError);
});
it('marks evals failed when the LLM is down', async () => {
  const s = await prepareEvals(me, { pack, baseline: 'BASE', axes, llm: fakeClient(() => new Error('down')), rng: seededRng(1), others: [] });
  expect(s.evalStatus).toBe('failed');
  expect(s.evals).toEqual([]);
});
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx vitest run tests/server/evalDrafts.test.ts`
Expected: FAIL, cannot find module

- [ ] **Step 3: Implement `src/server/evalDrafts.ts`**

The three drafts are generated in parallel. Each retries on an LLM error or a failed `checkNumbers`, up to `maxTries`. Slots are assigned by `shuffle([0, 1, 2], rng)`.

- [ ] **Step 4: Run tests to verify they pass**

Run: `npx vitest run tests/server/evalDrafts.test.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add poc/content/baseline-prompt.txt poc/src/server/evalDrafts.ts poc/tests/server/evalDrafts.test.ts
git commit -m "feat(poc): A/B/C eval drafts with retries"
```

---

### Task 9: Session store and engine

**Files:**
- Create: `poc/src/server/errors.ts`, `poc/src/server/store.ts`, `poc/src/server/engine.ts`
- Test: `poc/tests/server/engine.test.ts`

**Interfaces:**
- Consumes: Tasks 1–8.
- Produces, in `errors.ts`: `NotFoundError`, `ValidationError`, `ConflictError`, `OrderError` (each `extends Error`).
- Produces, in `store.ts`: `class FileStore { constructor(dir: string); save(s: Session): void; load(id: string): Session | null; list(): Session[]; logEvent(sessionId: string, type: string, data: unknown): void }`. Sessions go in `{dir}/sessions/{id}.json`. Events append to `{dir}/events.jsonl` as `{ at, sessionId, type, data }`.
- Produces, in `engine.ts`:

```ts
interface EngineDeps { store: FileStore; bank: Bank; pack: FactPack; axes: AxesConfig; llm: LlmClient; baseline: string; rng: Rng; now: () => string }
class Engine {
  constructor(deps: EngineDeps);
  start(input: { participantId: string; mode: Mode; retestOf?: string }): Session;
  currentRound(id: string): { round: Round; total: number } | null;     // null when all rounds answered
  recordPick(id: string, n: number, choice: Slot | 'none', chips: ReasonChip[], latencyMs: number): Session;
  evalView(id: string): { status: EvalStatus; trials?: Array<{ index: 0 | 1; cards: Array<{ slot: Slot; text: string }> }> };  // index 0 = E1, 1 = E2
  recordEval(id: string, index: 0 | 1, best: Slot, worst: Slot): Session;
  recordWorkload(id: string, w: Workload): Session;
  profileView(id: string): { summary: string; rules: Array<{ axis: Axis; text: string }> };
  recordRuleFeedback(id: string, axis: Axis, vote: 'up' | 'down'): Session;
  finish(id: string): Session;
  retryEvals(id: string): void;
  whenEvalsSettled(id: string): Promise<void>;
}
```

Behaviour the tests pin:
- `start` rejects a bad `participantId` (`ValidationError`) and a duplicate one (`ConflictError`).
- `currentRound` creates the next scheduled round only when the last one has a pick. Otherwise it returns the same open round. Cards come from `pickCards` with `currentLean(scoreRounds(rounds).votes)`, or `repeatCards` for the repeat round.
- `recordPick`: a wrong `n` gives `ConflictError`. The same choice repeated is a no-op. A different second choice gives `ConflictError`. A bad slot or chip gives `ValidationError`.
- After the last scored round (6 full, 4 short), the profile gets built, `evalStatus` becomes `'pending'`, and `prepareEvals` starts without being awaited. After the repeat round, `profile.consistent` is set.
- `recordEval` rejects `best === worst` (`ValidationError`). `profileView` throws `OrderError` until both trials have best and worst.
- Every state change calls `logEvent` with one of: `start`, `round_shown`, `pick`, `evals_ready`, `evals_failed`, `eval_choice`, `workload`, `rule_feedback`, `finish`.

- [ ] **Step 1: Write the failing tests**

```ts
// setup: temp dir, fixtureBank(), fixturePack(), axes, fake LLM answering 'The study enrolled 412 patients.', seededRng(1)
it('rejects names and duplicates', () => {
  expect(() => engine.start({ participantId: 'Jane Doe', mode: 'full' })).toThrow(ValidationError);
  engine.start({ participantId: 'W01', mode: 'full' });
  expect(() => engine.start({ participantId: 'W01', mode: 'full' })).toThrow(ConflictError);
});
it('serves the same open round until it is answered', () => {
  const s = engine.start({ participantId: 'W01', mode: 'full' });
  expect(engine.currentRound(s.id)).toEqual(engine.currentRound(s.id));
  expect(engine.currentRound(s.id)!.total).toBe(7);
});
it('same pick twice is a no-op, a different one conflicts', () => {
  const s = engine.start({ participantId: 'W01', mode: 'full' });
  engine.currentRound(s.id);
  engine.recordPick(s.id, 1, 0, [], 900);
  expect(() => engine.recordPick(s.id, 1, 0, [], 900)).not.toThrow();
  expect(() => engine.recordPick(s.id, 1, 2, [], 900)).toThrow(ConflictError);
  expect(() => engine.recordPick(s.id, 3, 0, [], 900)).toThrow(ConflictError);
});
it('resumes after restart', () => {
  const s = engine.start({ participantId: 'W01', mode: 'full' });
  const open = engine.currentRound(s.id);
  const engine2 = new Engine({ ...deps, store: new FileStore(dir) });
  expect(engine2.currentRound(s.id)).toEqual(open);
});
it('builds the profile after round 6 and prepares evals', async () => {
  const s = runRounds(engine, 'W01', 'full', 6);
  expect(store.load(s.id)!.profile).not.toBeNull();
  await engine.whenEvalsSettled(s.id);
  expect(engine.evalView(s.id).status).toBe('ready');
  expect(engine.evalView(s.id).trials!.map(t => t.index)).toEqual([0, 1]);
});
it('records consistency after the repeat round', () => {
  const s = runRounds(engine, 'W01', 'full', 7);
  expect(typeof store.load(s.id)!.profile!.consistent).toBe('boolean');
  expect(engine.currentRound(s.id)).toBeNull();
});
it('hides the profile until both eval choices are in', async () => {
  const s = runRounds(engine, 'W01', 'full', 7);
  await engine.whenEvalsSettled(s.id);
  expect(() => engine.profileView(s.id)).toThrow(OrderError);
  expect(() => engine.recordEval(s.id, 0, 1, 1)).toThrow(ValidationError);
  engine.recordEval(s.id, 0, 0, 2); engine.recordEval(s.id, 1, 1, 0);
  expect(engine.profileView(s.id).summary).toMatch(/^Your/);
});
it('retryEvals recovers after a failure', async () => { /* llm fails until a flag flips; retryEvals → 'ready' */ });
it('logs events', () => { /* events.jsonl contains start, round_shown, pick */ });
```
(`runRounds` is a test helper that starts a session and picks slot 0 for `k` rounds.)

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx vitest run tests/server/engine.test.ts`
Expected: FAIL, cannot find module

- [ ] **Step 3: Implement `errors.ts`, `store.ts` and `engine.ts`** to the interfaces above. Writes use `writeFileSync` to a temp file, then `renameSync`, so a crash never leaves half a session file.

- [ ] **Step 4: Run tests to verify they pass**

Run: `npx vitest run tests/server/engine.test.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add poc/src/server/errors.ts poc/src/server/store.ts poc/src/server/engine.ts poc/tests/server/engine.test.ts
git commit -m "feat(poc): session store and engine"
```

---

### Task 10: HTTP API and server startup

**Files:**
- Create: `poc/src/server/app.ts`, `poc/src/server/index.ts`
- Test: `poc/tests/server/app.test.ts`

**Interfaces:**
- Consumes: `Engine` and errors (Task 9), `bankReady` (Task 7), `clientFromEnv` (Task 6).
- Produces:
  - `createApp(engine: Engine, opts: { facilitatorToken: string; staticDir?: string }): express.Express`
  - `assertBankReady(bank: Bank): void` throws with every problem listed
  - `index.ts` reads `PORT` (3000), `DATA_DIR` (`./data`), `BANK_PATH` (`content/bank.json`), `BASELINE_PROMPT_PATH` (`content/baseline-prompt.txt`), `FACILITATOR_TOKEN` (required), and the LLM vars. It calls `assertBankReady`, then serves the API, `dist/client`, and `dist/client/index.html` for any other non-`/api` GET (so `/facilitator` loads).

| Method and path | Body | Response |
|---|---|---|
| `POST /api/sessions` | `{ participantId, mode, retestOf? }` | `201 { id }` |
| `GET /api/sessions/:id/round` | | `{ n, total, cards: [{ slot, text }] }` or `{ done: true }` |
| `POST /api/sessions/:id/rounds/:n/pick` | `{ choice: 0\|1\|2\|"none", chips, latencyMs }` | `200 {}` |
| `GET /api/sessions/:id/eval` | | `{ status }`, plus `trials: [{ index, cards: [{ slot, text }] }]` when ready |
| `POST /api/sessions/:id/eval/:index` | `{ best, worst }` (slots) | `200 {}` |
| `POST /api/sessions/:id/workload` | six numbers | `200 {}` |
| `GET /api/sessions/:id/profile` | | `{ summary, rules: [{ axis, text }] }` |
| `POST /api/sessions/:id/rule-feedback` | `{ axis, vote }` | `200 {}` |
| `POST /api/sessions/:id/finish` | | `200 {}` |
| `POST /api/facilitator/sessions/:id/retry-evals?token=` | | `202 {}` |

Errors map to `NotFoundError` 404, `ValidationError` 400, `ConflictError` 409 and `OrderError` 409, each with body `{ error: message }`. A missing or wrong `token` gives 401.

- [ ] **Step 1: Write the failing tests**

```ts
it('creates a session and rejects duplicates', async () => {
  expect((await request(app).post('/api/sessions').send({ participantId: 'W01', mode: 'full' })).status).toBe(201);
  expect((await request(app).post('/api/sessions').send({ participantId: 'W01', mode: 'full' })).status).toBe(409);
});
it('round response never reveals variant IDs', async () => {
  const body = (await request(app).get(`/api/sessions/${id}/round`)).body;
  expect(body.total).toBe(7);
  expect(body.cards).toHaveLength(3);
  expect(JSON.stringify(body)).not.toMatch(/P[123]-D[01]S[01]O[01]H[01]/);
});
it('eval response never reveals labels or prompts', async () => {
  // run 7 rounds, await engine.whenEvalsSettled(id)
  const s = JSON.stringify((await request(app).get(`/api/sessions/${id}/eval`)).body);
  expect(s).not.toMatch(/"label"|"system"|"A"|"B"|"C"/);
});
it('profile is refused before both eval choices', async () =>
  expect((await request(app).get(`/api/sessions/${id}/profile`)).status).toBe(409));
it('wrong round number conflicts', async () =>
  expect((await request(app).post(`/api/sessions/${id}/rounds/3/pick`).send({ choice: 0, chips: [], latencyMs: 1 })).status).toBe(409));
it('facilitator routes need the token', async () => {
  expect((await request(app).post(`/api/facilitator/sessions/${id}/retry-evals`)).status).toBe(401);
});
it('refuses to start with an unready bank', () =>
  expect(() => assertBankReady(fixtureBank().slice(1))).toThrow(/missing P1-D0S0O0H0/));
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx vitest run tests/server/app.test.ts`
Expected: FAIL, cannot find module

- [ ] **Step 3: Implement `app.ts` and `index.ts`** to the table above.

- [ ] **Step 4: Run tests to verify they pass**

Run: `npx vitest run tests/server && npm run typecheck`
Expected: PASS, then exit 0

- [ ] **Step 5: Commit**

```bash
git add poc/src/server/app.ts poc/src/server/index.ts poc/tests/server/app.test.ts
git commit -m "feat(poc): HTTP API and startup checks"
```

---

### Task 11: Participant screens

**Files:**
- Create: `poc/vite.config.ts`, `poc/playwright.config.ts`
- Create: `poc/src/client/index.html`, `main.ts`, `api.ts`, `style.css`, and `screens/setup.ts`, `round.ts`, `evalScreen.ts`, `workload.ts`, `profile.ts`
- Create: `poc/tests/fixtures/write-bank.ts`
- Test: `poc/tests/e2e/session.spec.ts`

**Interfaces:**
- Consumes: the HTTP API (Task 10) only. The client imports nothing from `src/server`.
- `vite.config.ts`: `root: 'src/client'`, `build.outDir: '../../dist/client'`, and in dev, proxy `/api` to `http://localhost:3000`.
- `playwright.config.ts`: `webServer` runs `npm run build && tsx tests/fixtures/write-bank.ts && tsx src/server/index.ts` with `LLM_FAKE=1`, `BANK_PATH=.e2e-data/bank.json`, `DATA_DIR=.e2e-data`, `FACILITATOR_TOKEN=test`, and port 3000. `write-bank.ts` writes `fixtureBank(160)` to `.e2e-data/bank.json`. The viewport is 1280×800.

**Screen copy (exact)**

| Screen | Copy |
|---|---|
| Setup | Title `Drafting preferences`; fields `Participant ID`, `Mode` (`Full (7 rounds)` / `Short (5 rounds)`), `Retest of (optional)`; button `Start` |
| Round | Heading `Round {n} of {total}`; prompt `Which version would you rather start from?`; button `None of these`; after a pick: `Why? (optional)`, the six chips, button `Next` |
| Waiting | `Preparing the next step…` (poll `/eval` every 2 s); on `failed`: `Something went wrong. Please tell the facilitator.` |
| Eval | `Draft {i} of 2`; first `Which draft is best?`, then `Which draft is worst?` (the best card is disabled) |
| Workload | `How did that feel?`, then six sliders with the standard NASA-TLX item names and questions, 0–100, button `Continue` |
| Profile | Heading `Your style`; the summary; each rule with `Fits me` / `Doesn't fit`; button `Finish` |
| Done | `Thanks. You're done.` |

Latency runs from cards rendered to card clicked. Picking a card highlights it and shows the chips. `Next` sends the pick. `None of these` shows the chips too.

- [ ] **Step 1: Write the failing e2e tests**

```ts
test('full run from setup to done', async ({ page }) => {
  await page.goto('/');
  await page.getByLabel('Participant ID').fill('E2E01');
  await page.getByRole('button', { name: 'Start' }).click();
  for (let n = 1; n <= 7; n++) {
    await expect(page.getByRole('heading', { name: `Round ${n} of 7` })).toBeVisible();
    await page.locator('[data-card]').first().click();
    await page.getByRole('button', { name: 'Next' }).click();
  }
  for (let i = 1; i <= 2; i++) {
    await expect(page.getByText(`Draft ${i} of 2`)).toBeVisible();
    await page.locator('[data-card]').nth(0).click();
    await page.locator('[data-card]').nth(2).click();
  }
  await page.getByRole('button', { name: 'Continue' }).click();
  await expect(page.getByRole('heading', { name: 'Your style' })).toBeVisible();
  await page.getByRole('button', { name: 'Finish' }).click();
  await expect(page.getByText("Thanks. You're done.")).toBeVisible();
});
test('reload keeps the round', async ({ page }) => { /* answer 2 rounds, reload, see "Round 3 of 7" */ });
test('three 160-word cards fit without scrolling', async ({ page }) => {
  /* start a session, then: */
  expect(await page.evaluate(() => document.documentElement.scrollHeight <= window.innerHeight)).toBe(true);
});
test('short mode has 5 rounds', async ({ page }) => { /* pick Short, see "Round 1 of 5" */ });
```
The session ID lives in `location.hash` (`#s={id}`), so a reload can resume.

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx playwright test`
Expected: FAIL, no `Participant ID` field

- [ ] **Step 3: Implement the client**

This is plain DOM code, one module per screen, with `main.ts` choosing the screen from the server's state. Use the layout picked in build-plan Step 4 (pstack `prototype` playbook). If that step hasn't happened yet, use three equal columns.

- [ ] **Step 4: Run tests to verify they pass**

Run: `npx playwright test`
Expected: 4 passed

- [ ] **Step 5: Commit**

```bash
git add poc/vite.config.ts poc/playwright.config.ts poc/src/client poc/tests/e2e poc/tests/fixtures/write-bank.ts
git commit -m "feat(poc): participant screens"
```

---

### Task 12: Analysis, facilitator view, CSV export

**Files:**
- Create: `poc/src/core/analysis.ts`, `poc/src/client/screens/facilitator.ts`
- Modify: `poc/src/server/app.ts` (two routes), `poc/src/client/main.ts` (route `/facilitator`)
- Test: `poc/tests/core/analysis.test.ts`, `poc/tests/server/app.test.ts` (two more tests)

**Interfaces:**
- Consumes: Task 1 types, `wordCount` (Task 2).
- Produces:

```ts
interface Count { wins: number; trials: number }
interface Analysis {
  people: number;
  consistent: Count;                 // wins = consistent true; trials = consistent not null
  bOverA: Count;                     // excludes bEqualsA trials
  bOverCReal: Count; bOverCMirror: Count;
  noneRate: Count;                   // wins = 'none' picks; trials = answered scored rounds
  slotPicks: [number, number, number]; slotBias: boolean;   // bias = one slot > 50% of non-none picks
  bestWasShortest: Count; bestWasB: Count;                  // ties for shortest count as shortest
  oppositePairs: Count;              // pairs with opposite FIRM rules on at least one axis
  retests: Array<{ participantId: string; retestOf: string; sameAxes: number }>;  // same pole or both none, out of 4
  tlxMedian: Workload | null;
}
analyze(sessions: Session[]): Analysis
toCsv(sessions: Session[]): string
```
Rank is best = 1, worst = 3, the other = 2. "B over A" means rank(B) < rank(A).
The CSV header is exactly `participantId,mode,retestOf,D,S,O,H,rules,consistent,noneCount,e1Best,e1Worst,e2Best,e2Worst,cSource,bEqualsA,tlxMental,tlxPhysical,tlxTemporal,tlxPerformance,tlxEffort,tlxFrustration`.
- Routes: `GET /api/facilitator/sessions?token=` returns `{ sessions, analysis }`, and `GET /api/facilitator/export.csv?token=` returns `text/csv`.
- The facilitator screen at `/facilitator?token=…` shows a table with one row per person (axis scores, rules, consistent, None count, eval picks with labels, TLX), the `Analysis` counts as `wins of trials`, a `Retry drafts` button for failed sessions, and an `Export CSV` link.

- [ ] **Step 1: Write the failing tests**

```ts
it('B over A uses full rank and skips bEqualsA', () => {
  const a = analyze([trialSession('W01', { best: 'B', worst: 'A' }), trialSession('W02', { best: 'C', worst: 'A' }),
                     trialSession('W03', { best: 'A', worst: 'B' }), trialSession('W04', { best: 'B', worst: 'C', bEqualsA: true })]);
  expect(a.bOverA).toEqual({ wins: 2, trials: 3 });
});
it('splits C comparisons by source', () => { /* cSource 'mirror' counts only in bOverCMirror */ });
it('flags slot bias above half', () => {
  expect(analyze([picksAtSlots([0, 0, 0, 0, 0, 0, 1, 1, 2, 2])]).slotBias).toBe(true);
  expect(analyze([picksAtSlots([0, 0, 0, 0, 0, 1, 1, 1, 2, 2])]).slotBias).toBe(false);
});
it('counts opposite firm rules only', () => {
  expect(analyze([withRules('X', [{ axis: 'D', pole: 0, strength: 'firm' }]), withRules('Y', [{ axis: 'D', pole: 1, strength: 'firm' }])]).oppositePairs).toEqual({ wins: 1, trials: 1 });
  expect(analyze([withRules('X', [{ axis: 'D', pole: 0, strength: 'firm' }]), withRules('Y', [{ axis: 'D', pole: 1, strength: 'soft' }])]).oppositePairs).toEqual({ wins: 0, trials: 1 });
});
it('compares retests axis by axis', () => { /* W01 vs W01-2 same on D,S → sameAxes 2 */ });
it('writes the CSV header', () => expect(toCsv([]).split('\n')[0]).toBe('participantId,mode,retestOf,D,S,O,H,rules,consistent,noneCount,e1Best,e1Worst,e2Best,e2Worst,cSource,bEqualsA,tlxMental,tlxPhysical,tlxTemporal,tlxPerformance,tlxEffort,tlxFrustration'));
// app.test.ts
it('facilitator sessions need the token', async () => {
  expect((await request(app).get('/api/facilitator/sessions')).status).toBe(401);
  expect((await request(app).get('/api/facilitator/sessions?token=test')).status).toBe(200);
});
it('exports CSV', async () =>
  expect((await request(app).get('/api/facilitator/export.csv?token=test')).headers['content-type']).toMatch(/text\/csv/));
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx vitest run tests/core/analysis.test.ts tests/server/app.test.ts`
Expected: FAIL, cannot find module `analysis`

- [ ] **Step 3: Implement `analysis.ts`, the two routes, and the facilitator screen.**

- [ ] **Step 4: Run tests to verify they pass**

Run: `npm test`
Expected: all PASS

- [ ] **Step 5: Commit**

```bash
git add poc/src/core/analysis.ts poc/src/server/app.ts poc/src/client poc/tests
git commit -m "feat(poc): analysis, facilitator view and CSV export"
```

---

### Task 13: Persona dry-run script

**Files:**
- Create: `poc/src/scripts/persona-run.ts`
- Test: `poc/tests/server/persona.test.ts`

**Interfaces:**
- Consumes: the HTTP API (Task 10), `LlmClient` (Task 6).
- Produces:
  - `parseChoice(reply: string): Slot | 'none' | null` (`"1"` means slot 0)
  - `runPersona(opts: { baseUrl: string; participantId: string; persona: string; mode: Mode; llm: LlmClient }): Promise<string>` returns the session ID
  - CLI: `npm run persona -- --id P-terse --persona "..." [--mode short] [--url http://localhost:3000]`

Persona prompts: system `You are a medical writer who {persona}. Answer with 1, 2, 3, or NONE only.` The round user prompt is `Which paragraph would you rather start from?\n\n1. {text}\n\n2. {text}\n\n3. {text}`. Eval uses `Which draft is best?` and then `Which draft is worst?` with the same layout. Workload is 50 for every item. Rule feedback is skipped.

- [ ] **Step 1: Write the failing tests**

```ts
it('parses replies', () =>
  expect(['1', ' 3.', 'NONE', 'Option 2', 'four'].map(parseChoice)).toEqual([0, 2, 'none', 1, null]));
it('drives a whole session', async () => {
  // start createApp on an ephemeral port with the fake engine
  const llm = fakeClient(req => (req.user.startsWith('Which draft is worst') ? '3' : '1'));
  const id = await runPersona({ baseUrl, participantId: 'P-test', persona: 'likes short text', mode: 'full', llm });
  const s = store.load(id)!;
  expect(s.rounds.filter(r => r.pick !== null)).toHaveLength(7);
  expect(s.finishedAt).not.toBeNull();
});
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `npx vitest run tests/server/persona.test.ts`
Expected: FAIL, cannot find module

- [ ] **Step 3: Implement `persona-run.ts`.** If a reply doesn't parse, ask again once, then send `none`.

- [ ] **Step 4: Run tests to verify they pass**

Run: `npx vitest run tests/server/persona.test.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add poc/src/scripts/persona-run.ts poc/tests/server/persona.test.ts
git commit -m "feat(poc): persona dry-run script"
```

---

### Task 14: Acceptance run (no new code)

This matches build-plan Steps 7 and 12–15. Each step is a command and the result that means it passed.

- [ ] **Step 1: Everything green**

Run: `npm run typecheck && npm test && npx playwright test`
Expected: exit 0, all tests pass

- [ ] **Step 2: Real LLM works**

Run: `ANTHROPIC_API_KEY=… LLM_MODEL=… npm run llm:smoke`
Expected: one or two sentences about weight and BMI that only use numbers from E1

- [ ] **Step 3: Build the real bank**

Run: `npm run bank`, then `npm run bank -- --only <each failed id>` until nothing is listed
Expected: `content/bank.json` holds 48 texts and the failed list is empty

- [ ] **Step 4: Human review**

Run: `npm run bank:export-review`. A medical writer who won't be a participant (open question 13) fills `humanOk` and `note` in `content/bank-review.csv`. Run `npm run bank:import-review`, regenerate any `no` rows, and repeat.
Run: `npm run bank:check`
Expected: `Bank ready: 48 texts`

- [ ] **Step 5: Run it yourself**

Run: `npm run build && FACILITATOR_TOKEN=… npm start`, then do one full and one short session in a browser.
Expected: both reach `Thanks. You're done.`, both `data/sessions/*.json` files read correctly by eye, and `/facilitator?token=…` shows both

- [ ] **Step 6: Persona dry run**

Run: `npm run persona -- --id P-terse --persona "wants short, terse text with no fluff and only the key numbers"` and `npm run persona -- --id P-full --persona "wants full explanations and every number reported"`
Expected: on the facilitator view, P-terse has pole 0 rules and P-full has pole 1 rules on D and H. This only proves the plumbing (build-plan §10, Step 13).

- [ ] **Step 7: Review**

Use `superpowers/skills/requesting-code-review` on the whole `poc/` diff. Fix every finding or write down why not.

- [ ] **Step 8: Commit**

```bash
git add poc/content/bank.json poc/content/bank-review.csv
git commit -m "chore(poc): reviewed paragraph bank"
```
