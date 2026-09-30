# ATS & Recruiter Match Score — 100-point rubric

Always report the breakdown, never just the total. Score the same way every time so the user can compare versions.

| # | Dimension | Points | How to score |
|---|---|---|---|
| 1 | **Keyword coverage** | 25 | (must-have JD keywords present ÷ total must-have keywords) × 25. Must-have = 15–20 phrases extracted in the gap analysis. Verbatim or exact-synonym match only. |
| 2 | **Keyword placement** | 10 | 10 if ≥ 80% of must-haves appear in the top third (headline + summary + skills + first role); 6 if 60–79%; 3 if 40–59%; 0 below. |
| 3 | **Title alignment** | 5 | 5 if the JD's title (or exact equivalent) appears in the headline; 3 if it appears elsewhere; 0 if absent. |
| 4 | **Parse safety** | 10 | Start at 10. −3 for any table/column/text box, −2 for header/footer content, −2 for non-standard section names, −1 for non-standard fonts, −2 for icons/graphics/photo (where market disallows). |
| 5 | **Quantification** | 15 | (bullets containing a number ÷ total bullets) × 15. Target ≥ 60%. |
| 6 | **Bullet quality** | 10 | Start at 10. −1 per fluff verb ("responsible for", "helped", "worked on", "involved in"), −1 per passive sentence, −1 per bullet over 2 lines, −1 per repeated opening verb in consecutive bullets. Floor 0. |
| 7 | **Scope & progression** | 10 | 10 if every role in the last 10 years shows scope (team/volume/scale) AND numbers grow over time; 6 if scope present but flat; 3 if scope only in current role; 0 if none. |
| 8 | **Length & layout** | 5 | Standard is 2 pages maximum. 5 for 1–2 pages; 2 for exactly one page over the max (3 pages at the default); 0 for two or more pages over. The web scorer renders the résumé at real Letter/A4 page size to count exactly; the CLI estimates from line count (~55 lines/page) unless `--pages` is given. |
| 9 | **Humanization** | 5 | Start at 5. −1 per banned cliché (spearheaded, leveraged, synergy, seamlessly, passionate, results-driven, proven track record, dynamic, cutting-edge); −2 if summary is generic enough to belong to anyone. Floor 0. |
| 10 | **Hygiene** | 5 | 5 if zero typos, consistent dates/tense/punctuation; −1 per issue; floor 0. |

## Interpreting the total

| Score | Meaning | Action |
|---|---|---|
| 85–100 | Shortlist-ready | Submit. Focus remaining effort on interview prep. |
| 70–84 | Competitive but beatable | Fix the two lowest dimensions before submitting. |
| 50–69 | Likely filtered out in large pools | Retailor keywords; add numbers; restructure top third. |
| < 50 | Will not pass ATS | Rebuild from the pipeline start. |

## Reporting format

```text
## ATS & recruiter match: 82 / 100
Keyword coverage 21/25 · Placement 10/10 · Title 5/5 · Parse safety 10/10
Quantification 9/15 · Bullet quality 8/10 · Scope & progression 8/10
Length & layout 5/5 · Humanization 4/5 · Hygiene 2/5

**Lowest dimensions:** Quantification (5 bullets lack numbers — see list), Hygiene (3 date-format inconsistencies)
**Missing keywords:** Terraform, CloudWatch, "root cause analysis"
```

## Caveats to state once

This estimates how a typical keyword-matching ATS and an experienced recruiter would rate the resume. Real ATS platforms (Workday, Taleo, Greenhouse, SuccessFactors, iCIMS) weight fields differently and most do not expose a score; recruiter-set knockout questions can override any match. Never present the number as a guarantee.
