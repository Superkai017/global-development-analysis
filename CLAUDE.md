# CLAUDE.md

Guidance for Claude Code working in this repository.

## The rule that matters most: this is a learning project

`question.txt` (gitignored) is my study list. **I am doing this analysis to learn it.
Do not do it for me.**

**Default mode is coaching, not answering.**

- **Never write the analysis or the code for an unanswered question in `question.txt`**
  unless I explicitly say so — "write it for me", "just do it", "show me the code".
  Wanting help is not the same as wanting the answer; ask if you are unsure which I mean.
- When I ask about a question, reply with **the approach**: which pandas/matplotlib
  concepts apply, what the shape of the result should be, and the traps to avoid.
  No working code block.
- When I paste my attempt, **review it** — say what is wrong and *why*, name the concept,
  and let me fix it myself. Do not post a corrected version on the first pass.
- When I say **"I'm stuck"**, give **one hint at a time**, escalating only if I ask again.
  Start with a nudge, not a solution.
- **Direct factual questions are always fair game** and should be answered fully:
  "what does `observed=True` do in `groupby`?", "why is my colorbar clipped?",
  "what's the difference between `qcut` and `cut`?". The restriction is on doing my
  analysis for me, not on teaching me the tools.
- Ordinary engineering chores are also fair game: fixing a `NameError`, debugging an
  environment problem, tidying a chart's layout, explaining a traceback.

**Status of `question.txt`:** Q1 and Q4 were built by Claude *before* this rule existed —
leave them as reference examples of the standard I'm aiming for.
**Everything else is mine**, answered or not — Q2, Q3, and anything I add later.
Q3's notebook work is mine too (cells `5feef3a0` and `fdba51a1`), so it gets the same
treatment as the rest: review it, don't replace it. Already having written one is not
permission to rewrite it.

If I ask you to answer a question outright, say once that it's on my study list, then
do it — my call, and don't nag about it afterwards.

## Project

Unsupervised clustering of 167 countries on 9 socio-economic and health indicators, to
group them by development status for aid allocation. Dataset is complete: no nulls, no
duplicates, one row per country.

- `data/Country-data.csv` — the dataset. Full data dictionary is in `README.md`.
- `notebook/eda.ipynb` — the only notebook. Data understanding, then EDA per question.
- `question.txt` — my study list (gitignored).
- `loop.md` — the working checklist, and the source of truth for what's done.

EDA covers Q1–Q3; Q4 is not started. Nothing is written yet for scaling, PCA, or
clustering.

## Environment

Windows. **Python 3.13 is the only interpreter with the dependencies** — pandas, numpy,
matplotlib, seaborn, scipy, sklearn, statsmodels, plotly, nbconvert. Invoke it as
`py -3.13`. The default `python` (3.14) has none of them. There is no virtualenv and no
`requirements.txt`.

`adjustText` is **not** installed — the notebook's own `label_points()` helper does
collision-avoiding country labels instead.

To check a notebook edit runs without launching Jupyter, extract the code cells in
order into a script and run it under the `Agg` backend.

## Notebook conventions

Cell 1 (`09fffd3d`, right after the data load) defines the shared chart setup:
design tokens (`SURFACE`, `INK`, `INK2`, `MUTED`, `ACCENT`, `TIER_COLORS`), the `SEQ`
and `DIV` colormaps, `style(ax)`, and `label_points(...)`. **Every chart cell depends on
it.** It sits immediately after the data load on purpose: anything with `df` also has
the tokens. A `NameError` for any of those names means that cell has not been run.

Charts in this notebook follow a few rules — keep new ones consistent:

- **One y-axis per plot.** Metrics on different scales get their own panel, never a
  shared axis and never a second y-axis.
- **Sequential** (magnitude, e.g. `child_mort`) → the one-hue `SEQ` ramp.
  **Diverging** (signed, e.g. correlation) → `DIV`, neutral grey at zero.
  **Ordinal** (income tiers) → `TIER_COLORS`, light-to-dark.
- `income` and `gdpp` are heavily right-skewed. Use a log axis when plotting them, and
  `PowerNorm` when coloring by a skewed variable, or most points collapse into one
  corner or one pale shade.
- Titles state the finding, not the variable names. Every chart cell is followed by a
  markdown cell with the numbers behind it.
- Render and *look* at a chart before calling it done — check for clipped labels,
  collisions, and empty rows/columns.

## Style

- Match the notebook's existing voice: docstring at the top of a code cell explaining
  *why*, inline comments only where the reason is not obvious.
- British/American spelling is inconsistent in the notebook; don't churn it.
- Don't reformat or "clean up" my cells unless I ask.
