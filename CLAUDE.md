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

**Status of `question.txt`:** Q1 and Q4 were built by Claude at my request, not by me —
Q1 before this rule existed, Q4 (cells `c006efd2`, `9f8b77e7`, `61c50476`) by asking for
it outright. Leave them as reference examples of the standard I'm aiming for.
**Everything else is mine**, answered or not — Q2, Q3, and anything I add later.
Q3's notebook work is mine too (cells `5feef3a0` and `fdba51a1`), so it gets the same
treatment as the rest: review it, don't replace it. Already having written one is not
permission to rewrite it.

`notebook/preprocessing.ipynb` is a mixed case: I wrote the first version, then asked
Claude to fix it (commit `0b1d2e4`), so the cells as they stand are largely Claude's.
That was a one-off for the transform bug, not a precedent — **`modelling.ipynb` is mine**:
PCA, K-Means, the dendrogram, the cluster profiling and the aid shortlist all get the
coaching treatment.

If I ask you to answer a question outright, say once that it's on my study list, then
do it — my call, and don't nag about it afterwards.

## Project

Unsupervised clustering of 167 countries on 9 socio-economic and health indicators, to
group them by development status for aid allocation. Dataset is complete: no nulls, no
duplicates, one row per country.

- `data/raw_data/Country-data.csv` — the dataset. Full data dictionary is in `README.md`.
- `data/preprocessed_data/preprocessed_data.csv` — the model matrix, 167 × 9, country as
  the index. Written by `preprocessing.ipynb`, tracked in git. Read it back with
  `pd.read_csv(path, index_col='country')` or country becomes a feature column.
- `notebook/eda.ipynb` — data understanding, then EDA per question. Owns the chart setup.
- `notebook/preprocessing.ipynb` — raw → transform → scale → save. Done.
- `notebook/modelling.ipynb` — PCA and clustering. Two cells so far: imports and the load.
- `question.txt` — my study list (gitignored).
- `preprocessplan.txt` — a pasted planning transcript, gitignored. Scratch, not a spec;
  `loop.md` overrides it wherever they disagree.
- `loop.md` — the working checklist, and the source of truth for what's done.

EDA covers Q1–Q4 and preprocessing is finished. Nothing is written yet for PCA or clustering.
`README.md`'s repo-structure block is stale — it still describes `src/`, a single
`notebooks/country_clustering.ipynb`, and the pre-restructure data path.

## Preprocessing — settled, don't reopen it

**Keep it simple.** The aim is clustering that's stable, not a wide engineered feature
set. Don't propose new derived variables unless I ask for them.

The pipeline is in `notebook/preprocessing.ipynb`: `df` (raw) → `X` (country as index) →
`X_log` → `X_yj` → `X_scaled` → CSV. Each stage is a new frame so the cells are
re-runnable — an in-place version silently gave `log(log(gdpp))` on a second run.

- `country` is an **ID only** — it never enters the model matrix. It's the index, not
  dropped, so cluster labels can be joined back to country names.
- **`income`, `gdpp` → `np.log`.** Skew 2.23 → −0.24 and 2.22 → 0.01. Both strictly
  positive; an `assert` guards that rather than a `0` fallback, because a `0` sentinel
  sorts *above* every real value once logged.
- **`inflation`, `exports`, `imports` → Yeo-Johnson** (`PowerTransformer`,
  `standardize=False`). Skew 5.15 → 0.18, 2.45 → 0.10, 1.91 → 0.17.
  - `inflation` is negative for 8 countries (Seychelles −4.21, Ireland −3.22, …), so
    `np.log` gives `NaN`; Yeo-Johnson is defined on the whole real line.
  - `exports`/`imports` are positive, so log *runs* — but Myanmar sits at 0.109 / 0.066
    against medians of 35 / 43, so log stretches the *bottom* tail and flips skew to
    −2.72 / −4.92. Leaving them raw was worse still: it made Euclidean distance measure
    trade openness, and k=3 returned a five-country "biggest exporters" cluster.
- **`child_mort`, `health`, `life_expec`, `total_fer` are left untransformed** — skew
  1.45 / 0.71 / −0.97 / 0.97, moderate enough not to bother.
- **`RobustScaler`, on all 9 features.** K-Means is Euclidean, so on raw units `income`
  (to 125,000) would drown `total_fer` (1.15–7.49). Robust over Standard because median
  and IQR aren't dragged by Haiti (child_mort 208), Sierra Leone (160) and Chad (150) the
  way mean and std are. **No country is capped or dropped** — the outliers are the point.
- `pt` and `scaler` are kept for `inverse_transform`, which is how a cluster centroid
  becomes a real percentage or dollar figure again. They only exist in that notebook's
  kernel, so profiling in `modelling.ipynb` has to re-fit or re-import them.

Q4 established the constraint these feed into: the 9 features are not 9 independent
signals. `child_mort`/`total_fer`/`life_expec` move as one block (|r| = 0.76–0.89) and
`income`/`gdpp` as another (0.90), so equal-weighted Euclidean distance is tilted toward
"need" before any decision is made about it.

## Environment

Windows. **Python 3.13 is the only interpreter with the dependencies** — pandas, numpy,
matplotlib, seaborn, scipy, sklearn, statsmodels, plotly, nbconvert. Invoke it as
`py -3.13`. The default `python` (3.14) has none of them. There is no virtualenv and no
`requirements.txt`.

`adjustText` is **not** installed — the notebook's own `label_points()` helper does
collision-avoiding country labels instead.

To check a notebook edit runs without launching Jupyter, extract the code cells in
order into a script and run it under the `Agg` backend.

Paths are inconsistent: `preprocessing.ipynb` uses `Path('..') / 'data' / …` relative to
`notebook/`, which is the version that survives the repo moving. `eda.ipynb` (`1ff645e7`)
and `modelling.ipynb` (`40941ec2`) still hardcode `D:\Country-development-analysis\…`.

## Notebook conventions

The chart setup lives in `eda.ipynb` only. `preprocessing.ipynb` and `modelling.ipynb`
don't have it, so a chart in either needs the tokens brought across first.

In `eda.ipynb`, cell 1 (`09fffd3d`, right after the data load) defines the shared setup:
design tokens (`SURFACE`, `INK`, `INK2`, `MUTED`, `ACCENT`, `TIER_COLORS`), the `SEQ`
and `DIV` colormaps, `style(ax)`, and `label_points(...)`. **Every chart cell depends on
it.** It sits immediately after the data load on purpose: anything with `df` also has
the tokens. A `NameError` for any of those names means that cell has not been run.

Charts follow a few rules in every notebook — keep new ones consistent:

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
