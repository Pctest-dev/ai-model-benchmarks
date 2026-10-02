# AI Model Comparison
## Standalone Run: Opus 5.5 vs Sonnet 5.5 vs GLM 5.3
*October 2026*

---

## Overview

On 29 September 2026, Anthropic's Frontier Red Team published a report on Z.ai's GLM-5.3, agreeing with NIST CAISI's assessment that it lags the US frontier by roughly four months. That figure is based on an aggregate of **cyber** benchmarks (exploit development).

This run asks a narrower question: does that gap show up on general coding and reasoning tasks? Three models were given the same three tests from the existing benchmark series, unchanged.

Models tested: **Claude Opus 5.5**, **Claude Sonnet 5.5**, **GLM 5.3**.

This is a standalone run. Results are not merged into the June 2026 leaderboards.

---

## Setup

- Identical prompt to every model, word for word, no modifications
- Single prompt, no follow-ups, no corrections
- Test 1 sites were checked in-browser (toggle, persistence, terminal input, edge cases)
- Test 2 scripts were executed as-is and fed additional edge cases (empty roster, all-empty scores, missing keys, ties, junk score values)
- Test 3 summaries were checked against the six known contradictions and gaps
- Five categories per test, 10 points each, 50 per test, 150 total

---

## Final Standings

| Rank | Model | Test 1 Web Dev | Test 2 Python | Test 3 Summarisation | Total /150 |
|---|---|---|---|---|---|
| 1 | Opus 5.5 | 46 | 49 | 48 | **143** |
| 2 | GLM 5.3 | 45 | 49 | 48 | **142** |
| 3 | Sonnet 5.5 | 46 | 45 | 48 | **139** |

**Headline:** GLM 5.3 finished one point behind Opus 5.5 and three ahead of Sonnet 5.5. Within single-run variance, this is effectively a three-way tie. The four-month cyber gap does not show up on these tasks.

---

## Test 1: Web Dev (Retro Robot Portfolio)

**Prompt:** Build a single-page portfolio site for a fictional retro robot in one HTML file with a nav bar, about section, skills grid, contact form, dark/light toggle with localStorage persistence, and a CRT-style terminal with at least 8 commands. No frameworks.

| Model | Robot | Correctness | Code Quality | Security | Look | Overall | Total |
|---|---|---|---|---|---|---|---|
| Sonnet 5.5 | TIN-9 | 10 | 9 | 10 | 8 | 9 | **46** |
| Opus 5.5 | RX-80 "Sprocket" | 9 | 9 | 10 | 9 | 9 | **46** |
| GLM 5.3 | BOLT-7 | 9 | 8 | 9 | 10 | 9 | **45** |

### Sonnet 5.5 (46)
Smallest and cleanest code of the three (26KB). 14 terminal commands, theme applied before first paint, `prefers-color-scheme` fallback, localStorage wrapped in try/catch. The only model to use `hasOwnProperty` on the terminal command lookup. Lost points on Look: clean and readable, but more "modern dark dev portfolio" than retro.

### Opus 5.5 (46)
Most feature-rich terminal: 13 commands including working `echo` with arguments, `history`, arrow-key recall and Ctrl+L. `echo` correctly repeats user input via `textContent`, which is exactly where several June models failed. Command lookup is a bare `commands[name]`, so typing `constructor` throws an uncaught TypeError and prints nothing.

### GLM 5.3 (45)
Best-looking site by a distance: BIOS boot text, spec chips, callsign contact panel, proper retro feel without tipping into ugly. Docked for code hygiene rather than capability:
- The main terminal `print()` helper writes via `innerHTML`. It is only fed canned strings, and user input goes through a separate safe `printText()`, so it is not exploitable as written, but it is a footgun for the next editor.
- After `shutdown`, a document-wide keydown listener reboots the terminal on any key and calls `preventDefault()`. The reboot-and-scroll-to-terminal is a deliberate (and funny) touch, but because it is page-wide, the first keystroke typed anywhere else (e.g. the contact form) is swallowed and keyboard navigation is blocked while the terminal is off.

**Test 1 note:** all three used `textContent` for user input. Nobody repeated the innerHTML XSS mistake that was endemic in June.

---

## Test 2: Python Data Processing

**Task:** Process a hardcoded student dataset: per-student averages, highest/lowest scorer, averages by subject, failing students (below 50), formatted terminal report. Handle edge cases like empty score lists. Standard library only.

**Expected:** Eve 91.25 highest, Frank 32.50 lowest, Maths 76.25 (excluding Judy), failing Bob 44.00 / Frank 32.50 / Heidi 47.50, Judy handled gracefully.

| Model | Correctness | Code Quality | Error Handling | Output | Overall | Total |
|---|---|---|---|---|---|---|
| GLM 5.3 | 10 | 9 | 10 | 10 | 10 | **49** |
| Opus 5.5 | 10 | 10 | 10 | 9 | 10 | **49** |
| Sonnet 5.5 | 10 | 8 | 9 | 9 | 9 | **45** |

### GLM 5.3 (49)
`build_report()` returns a string rather than printing as it goes, so it is testable. Ranked results table with status column, tie-aware top/bottom, exam counts per subject, and a warnings section explaining why Judy is excluded. Handled empty roster, all-empty scores, missing keys and a tie at exactly 50. Docked one on code quality for a nested ternary and a "clever" sort key that take a few reads to follow.

### Opus 5.5 (49)
Best structure: small, separate, documented functions for averages, extremes and subject grouping. Went further than asked by filtering junk scores (`None`, strings, and `True`, which Python treats as an int). Docked one on output: table is in input order rather than ranked, and fixed-width columns would misalign with longer names.

### Sonnet 5.5 (45)
Correct numbers and correct Judy handling, plus a nice touch showing raw scores in the table. Thinner under pressure:
- Everything lives in a single `main()`
- A misleading comment on the subject grouping contradicts what the code does
- Ties only report one student
- A record missing the `scores` key crashes with a KeyError
- An empty roster prints a blank table instead of a message
- Score column silently truncates at 15 characters

None of those cases are in the provided data, so deductions were kept small.

**Test 2 note:** all three returned `None` for Judy's empty score list. The `None` vs `0.0` trap that split the June field in half is solved at this tier. The differences here are depth of defensive coding, not basics.

---

## Test 3: Summarisation & Reasoning Under Ambiguity

**Prompt:** Summarise the following text. Where information is unclear, contradictory, or missing, explicitly flag it rather than picking one version or glossing over it. Do not make assumptions to fill gaps. Be concise.

**Six known issues:** start date (March vs April), budget (£240,000 vs £180,000), completion (December sign-off vs January letter), attendance (40% vs 12%), undisclosed funding breakdown, opposing councillors on the same committee.

| Model | Accuracy | Contradiction | Uncertainty | Conciseness | Hallucination | Total |
|---|---|---|---|---|---|---|
| Sonnet 5.5 | 9 | 10 | 10 | 9 | 10 | **48** |
| Opus 5.5 | 9 | 10 | 10 | 9 | 10 | **48** |
| GLM 5.3 | 9 | 10 | 9 | 10 | 10 | **48** |

All three flagged all six issues with explicit language. Nobody hallucinated.

### Sonnet 5.5 (48)
Category-labelled bullets. Went beyond the six: no baseline or time period for attendance, nature of outstanding issues unstated, no reasons given for either councillor's view, final cost never stated so overspend cannot be judged.

### Opus 5.5 (48)
Similar depth, plus the one catch nobody else made in either run: "last year" has no anchor date, so the actual year is unknown. Closing line summarising what *can* be said with confidence was useful. Lost a point on conciseness for an intro that repeats the bullets. Also wrote "renovated last year" when the source only says work *began* last year.

### GLM 5.3 (48)
Best structure: separate "Contradictions" and "Missing information" sections, essentially the `[CONTRADICTORY]` / `[MISSING]` split that topped June, done as headers. Tightest of the three. Did not go beyond the required gaps, and added slight editorial spin ("despite both sitting on the same committee").

**Test 3 note:** a genuine three-way tie, with points traded in different places. Sonnet and Opus dug deeper, GLM was tighter and better organised.

---

## Key Findings

### The four-month gap does not show up here
Anthropic and CAISI's four-month figure is about cyber exploit capability. On general web dev, Python and reasoning, GLM 5.3 sits one point behind Opus 5.5 out of 150. "Four months behind" should not be read as "worse at everything".

### Every model hardened the same hedge
The source says a January letter *suggests* outstanding issues remained. All three models, under explicit instructions to flag uncertainty, turned that into a firmer claim: the letter "says" (Sonnet, GLM) or "lists" (Opus) outstanding issues. Same slip, three models, two labs. Hedged language in a source is where models quietly tidy up, even when told not to.

### Single runs wobble
Sonnet 5.5 wrote the cleanest, most careful code in Test 1 and the thinnest in Test 2. Same model, opposite habits on consecutive tasks. A 1 to 4 point spread across 150 points is inside that noise.

### These tests are at ceiling for this tier
No Judy failures, no missed contradictions, no hallucinations, no XSS. Differences came down to details like a prototype lookup and one hardened verb. Harder tests (SQL, ESP32-S3 embedded C++) are needed to separate frontier models.

### Bigger model, bigger surface area
Opus 5.5 consistently built more features than Sonnet 5.5, and in Test 1 that extra surface is where its bug lived. Sonnet's smaller output in Test 1 was harder to break.

---

## Caveats

- **One run per test.** Repeat runs would likely shift scores by a few points either way.
- **Not directly comparable to June.** All three scored at or below their June predecessors (Sonnet 4.6, GLM 5.1/5.2). This most likely reflects stricter scoring this round (e.g. checking prototype lookups and hedged verbs) rather than weaker models. A re-score of the June entries with the same eye would settle it.
- **Agentic harnesses.** All models could execute and self-correct before submitting. Results will not match a plain web-chat run of the same prompt.
