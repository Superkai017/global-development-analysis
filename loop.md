# loop.md — project checklist

Working checklist for the country-clustering project. Tick a box when the work is in the
notebook it belongs to, the chart has been *looked at*, and the cell runs top-to-bottom clean.

Three notebooks now, one stage each: `notebook/eda.ipynb` (§1–2), `notebook/preprocessing.ipynb`
(§3), `notebook/modelling.ipynb` (§4–7).

Rule from `CLAUDE.md` still applies: the EDA questions and the modelling are mine to write.
Claude coaches — approach, review, one hint at a time — unless I explicitly ask for code.

---

## 1. Data understanding — done

- [x] Shape, dtypes, `describe()` on the 9 numeric features
- [x] 167 unique countries, no nulls, no duplicates
- [x] Shared chart setup cell (`09fffd3d`): tokens, `SEQ`/`DIV`, `style()`, `label_points()`

## 2. EDA — one question at a time

- [x] **Q1 (grouping)** — income tiers via `qcut`, compare `child_mort` / `life_expec` / `health`
  - [x] Tier summary table
  - [x] Three-panel chart (one y-axis per metric)
  - [ ] Markdown cell under the chart with the numbers behind it
- [ ] **Q2 (grouping)** — 15 countries with highest `child_mort` + lowest `gdpp`
  - [x] Decide how "combined" is defined (rank-sum? filter then sort? percentile score?)
    - currently a lexicographic sort: `child_mort` desc, `gdpp` asc. `gdpp` only breaks
      exact ties, so in practice this is top-15 by `child_mort` alone — revisit if the
      shortlist is meant to weigh both
  - [x] Result shape: 15 rows, country + both metrics, ordered by the combined criterion
  - [x] Chart or table — whichever actually communicates the shortlist
  - [ ] Markdown cell with the finding — prose is written, still needs the numbers behind it
- [x] **Q3 (visualization)** — `income` vs `life_expec`, coloured by `child_mort`
  - [x] Tier summary table: mean `life_expec` + `child_mort` per income tier
  - [x] Log x-axis for `income`; `PowerNorm(gamma=0.6)` on the colour scale (`child_mort` is skewed)
  - [x] Say whether it's linear or plateaus — the title calls it: steepest gains at low income
    - where the bend actually sits is still unquantified; a rough income figure would sharpen it
  - [ ] Markdown cell with the finding (`3b4fa3c5`) — prose is written and calls the plateau,
    still needs the numbers behind it, same gap as Q2
  - [ ] Silence the `observed=False` FutureWarning in the groupby (`5feef3a0`) — `income_tier`
    is categorical, so the default flips in a future pandas and changes the table's shape
    - still firing in the saved output; the tier means are 78.95 / 71.77 / 59.76 life_expec
      and 8.54 / 28.12 / 88.06 child_mort
- [x] **Q4 (visualization)** — `total_fer` vs `child_mort` + correlation heatmap of all 9 features
  - written by Claude at my request, not by me — read it before trusting it
  - [x] Scatter (`c006efd2`) with `label_points()` on `|residual|`, coloured by income tier
  - [x] Heatmap (`9f8b77e7`) on `DIV`, pinned `vmin=-1, vmax=1` so neutral grey lands on r = 0
  - [x] Markdown cell naming which variables carry the "need" signal (`61c50476`)
- [ ] Any new question added to `question.txt` gets its own block here

## 3. Preprocessing — done, in `notebook/preprocessing.ipynb`

Pipeline is `df` (raw) → `X` (country as index) → `X_log` → `X_yj` → `X_scaled`, each stage a
new frame so the cells are re-runnable. Output is `data/preprocessed_data/preprocessed_data.csv`,
167 × 9, no nulls, country as the index.

- [x] Confirm the feature matrix: 9 numeric columns, `country` held aside as the index/label
  - `X = df.set_index('country')` (`754a2bea`) — kept as the index rather than dropped so
    cluster labels can be joined back to country names, and it survives into the saved CSV
- [x] Right-skew in `income`, `gdpp` — **log-transformed** before scaling (`861167ac`)
  - skew 2.23 → −0.24 and 2.22 → 0.01. Both strictly positive, so plain `log` is safe; an
    `assert (X[LOG_COLS] > 0).all().all()` guards it instead of a `0` fallback (a `0` sentinel
    sorts *above* real values once logged)
- [x] `inflation` — **Yeo-Johnson**, not log (`ed29a86a`). Skew 5.15 → 0.18
  - 8 countries are negative (Seychelles −4.21, Ireland −3.22, Japan −1.90, …) so `np.log`
    gives `NaN`. Yeo-Johnson is defined on the whole real line — no shift needed
- [x] `exports` / `imports` — **Yeo-Johnson too**. Skew 2.45 → 0.10 and 1.91 → 0.17
  - these were the miss: logged `income`/`gdpp` spanned ±1.4 while raw `exports` spanned ±6.0,
    so Euclidean distance was mostly measuring trade openness. k=3 was pulling out Singapore,
    Luxembourg, Malta, Ireland, Seychelles as a 5-country "biggest exporters" cluster
  - plain log is *worse* here: Myanmar at 0.109 / 0.066 against medians of 35 / 43, so log
    stretches the bottom tail and flips skew to −2.72 / −4.92 (Myanmar at −9.75 scaled)
  - `standardize=False` on the `PowerTransformer` on purpose — `RobustScaler` does the
    centring next, and should see all 9 columns on the same footing
- [x] `child_mort`, `health`, `life_expec`, `total_fer` left untransformed
  - skew 1.45 / 0.71 / −0.97 / 0.97 — moderate, not worth a transform
- [x] Choose the scaler — **`RobustScaler`** (`db6589c9`), and the reason is written in the cell
  - median + IQR, so Haiti (child_mort 208), Sierra Leone (160) and Chad (150) don't inflate
    the spread and squash everyone else toward zero. `StandardScaler`'s mean *and* std are
    both dragged by exactly those points
  - `index=X_yj.index` passed explicitly, or the DataFrame constructor swaps country for a
    RangeIndex and the labels are gone
- [x] Outlier check — **keep every country, cap nothing**
  - Yeo-Johnson holds the extremes inside ±3.2 and `RobustScaler` absorbs the rest; capping
    would throw away exactly the countries the aid shortlist is about
- [x] Verification cell (`32002efc`) before the save: skew before/after, shape, nulls, index
- [x] Save to `data/preprocessed_data/preprocessed_data.csv` with `index=True` (`fa0e9923`)
- `pt` and `scaler` are kept, not discarded — `inverse_transform` is what turns a cluster
  centroid back into real percentages when the clusters need describing. They only live in
  the preprocessing kernel, so §7 profiling either re-fits them or moves into that notebook
- Constraint carried over from Q4: the 9 features are not 9 independent signals
  (`child_mort`/`total_fer`/`life_expec` at |r| = 0.76–0.89; `income`/`gdpp` at 0.90), so
  equal-weighted distance is still tilted toward "need" whatever the scaler does

## 4. Modelling notebook — K-Means and PCA in, runs clean

`notebook/modelling.ipynb` is 15 cells (13 code, 2 markdown labels): imports (`1e89ee29`),
load (`40941ec2`), the country split and `features` pin (`b6e1d698`), elbow (`ab0cb33a`),
metric table (`0d6c67c4`), metric charts (`037f92bd`), the k=3 fit (`eacd0c7a`), profile and
naming (`1ecfcb11`), per-cluster richest/poorest (`d189ff6b`), PCA 2D (`136bbda4`), PCA 3D
(`462f68ec`), per-cluster silhouette (`6223cf1d`), PC1 cross-check (`5fb276fe`).

Checked 2026-08-13: code cells extracted in order and run under `Agg` — exit 0, clean
top-to-bottom.

- [x] Country names survive the load — `b6e1d698` takes `countries = df['country'].copy()`
  before dropping the column, so `countries[mask]` names any subset of `X`. Not the
  `index_col='country'` route CLAUDE.md describes, but it solves the same problem: the Series
  and `X` both keep the original `RangeIndex`, and that alignment is what makes it work
- [x] `features` pinned once from `df.columns` in `b6e1d698`, and every fit and score takes
  `X[features]`. `X` accumulates `Cluster`, `PC1`, `PC2`, `PC3` as the notebook runs, so
  deriving the feature list from `X.columns` later silently fed those back in as model inputs
- [ ] Swap the absolute `D:\Country-development-analysis\…` path for a relative one, the way
  `preprocessing.ipynb` does (`Path('..') / 'data' / …`) — `eda.ipynb` (`1ff645e7`) has the
  same absolute path and the same fix
- [ ] `StandardScaler` / `PowerTransformer` are imported but nothing re-fits them — still dead
  imports. `PCA` has stopped being one (§5)
- [ ] No chart tokens in this notebook: `SURFACE`, `SEQ`, `style()`, `label_points()` all live
  in `eda.ipynb` cell `09fffd3d`. The four chart cells here use bare matplotlib defaults plus
  literal `'steelblue'` / `'salmon'` and seaborn's `Set2`
- [ ] Chart conventions not applied yet: titles name variables rather than state a finding
  ('Elbow Method — Inertia vs Number of Clusters', 'Country Segments — PCA 2D Projection'),
  and no chart has a markdown cell with the numbers under it — the two markdown cells present
  (`be49699f`, `29398e25`) are bare labels
- [x] Saved outputs are consistent — resolved by a Restart & Run All before commit `7139368`.
  Execution counts run 1–13 in order with no error outputs, and every cell now reports the
  same k=3 run (52 / 72 / 43). They previously mixed k=3 and k=4 results in one file

## 5. PCA — fitted, not yet read

- [x] Fit PCA on the scaled matrix — `136bbda4` (2 components) and `462f68ec` (3), both on
  `X[features]` so the `Cluster` column can't leak into the decomposition
- [ ] Scree / cumulative-variance plot, pick the number of components
  - the numbers are printed but never plotted: PC1 49.55%, PC2 18.69%, PC3 14.30%,
    cumulative 82.54%. n_components was chosen to be plottable (2D, then 3D), not from a scree
- [ ] Read the loadings — what does PC1 actually mean in development terms?
  - printed in both cells, not written up anywhere. PC1 is the "need" axis Q4 predicted:
    life_expec +0.435, child_mort −0.429, total_fer −0.402, income +0.366, gdpp +0.362 — the
    two correlated blocks loading together with opposite signs
  - PC2 is trade openness almost by itself (imports +0.696, exports +0.591), which is the same
    signal §3 spent the Yeo-Johnson work stopping from dominating Euclidean distance
  - PC3 is health +0.649 against inflation −0.636
- [ ] Decide: cluster on components or on the scaled features
  - currently clustering on all 9 scaled features and using PCA only to *display* the result.
    That's a defensible choice, but it is a choice and it isn't written down
- [ ] `136bbda4` and `462f68ec` both fit PCA and both write `PC1`/`PC2`. Harmless — components
  don't change with `n_components` — but one of the two should go

## 6. Clustering — K-Means at k=3, hierarchical not started

- [ ] K-Means: elbow (inertia) + silhouette to choose `k`
  - [x] Elbow over k = 2–10, `n_init=10`, `random_state=42`, on `X[features]` (`ab0cb33a`)
  - [x] All three metrics computed and tabled (`0d6c67c4`); silhouette and Davies-Bouldin
    plotted side by side (`037f92bd`)
  - [x] k = 3 fitted (`eacd0c7a`) — cluster sizes 52 / 72 / 43. Matches the ad-hoc k=3 run from
    the preprocessing debugging (43 / 72 / 52), which held across seeds
  - [x] re-runnable: fits on `X[features]`, so a second run can't cluster on `Cluster` or the
    PC columns the way it used to
  - [x] Per-cluster silhouette (`6223cf1d`) — means 0.252 / 0.209 / 0.256, minimums
    −0.004 / 0.000 / −0.060. Nothing badly misassigned; cluster 1, the 72-country middle, is
    the loosest, which is what you'd expect of a residual "everyone else" group
  - [ ] **Say which metric is deciding, and write down why k=3** — the table still doesn't
    hand it to you. Silhouette peaks at k=2 (0.3261 against 0.2345 at k=3), Davies-Bouldin is
    also best at k=2 (1.2203 against 1.4401), and Calinski-Harabasz falls monotonically. k=3 is
    defensible on interpretability — it maps onto least-developed / emerging / developed and
    the aid question needs more than a two-way split — but that argument has to be written
- [ ] Hierarchical: dendrogram, pick the cut, compare the grouping to K-Means
  - `scipy` is available but nothing is imported for it yet in `1e89ee29`
- [x] Lock in a final labelling and attach it to `df`
  - `1ecfcb11` orders clusters by `dev_score` ascending, zips that order to `tiers`, and
    attaches `df['Segment'] = X['Cluster'].map(segment)`. Alignment works because `X = df.copy()`
    keeps the same `RangeIndex` — it breaks the moment either frame is reindexed on country
  - an `assert len(tiers) == k` guards the zip: a short `tiers` list used to drop a cluster
    silently and leave `NaN`s in `Segment` (that's where the old `KeyError: 3` came from)

## 7. Cluster profiling & the actual answer — most of the way

- [ ] Mean of each of the 9 indicators per cluster
  - [x] `profile = X.groupby('Cluster')[features].mean().round(2)` with an `n` column (`1ecfcb11`)
  - [x] `dev_score` = gdpp + income + life_expec − child_mort − total_fer, used to rank the
    clusters least → most developed. Only legitimate because all 9 features share a scale after
    §3 — it would be meaningless on raw units
  - [ ] still in transformed units — `inverse_transform` back through the scaler and the
    `PowerTransformer` before quoting a number as a percentage or a dollar figure. Neither `pt`
    nor `scaler` exists in this kernel, so this notebook has to re-fit or re-import them
- [x] Name the clusters in plain language
  - Least developed (43), Emerging (72), Developed (52). Assigned by `dev_score` rank rather
    than hardcoded per cluster id, so they re-derive correctly if the seed or `k` changes
- [x] Scatter of the clusters on the two strongest axes, labelled
  - `136bbda4` in 2D on PC1/PC2, `462f68ec` in 3D on PC1/PC2/PC3, coloured by segment
  - [ ] points carry no country names — `label_points()` from `eda.ipynb` is the tool, and it
    needs the setup cell brought across first (§4)
- [x] Produce the aid shortlist — which countries fall in the neediest cluster
  - all 43 Least-developed countries are written out in `README.md`, ordered by child mortality
  - the cluster spans $231 – $3,600 gdpp, and its *lowest* child mortality is Nepal at 47.0 —
    2.4× the global median of 19.3, so it separates cleanly on the metric the question is about
  - [ ] it lives in the README, not in the notebook. `d189ff6b` still only prints the five
    richest and poorest per cluster — a cell that emits the ordered 43 belongs here too
- [x] Sanity check: does the shortlist agree with the Q2 shortlist? Explain any difference
  - **14 of Q2's 15 fall inside the Least-developed cluster** with no supervision
  - the one disagreement is **Equatorial Guinea** — 111 child deaths per 1,000 but $33,700
    income and $17,100 gdpp — which clusters as Emerging. Q2's lexicographic sort ranked it on
    mortality alone; the clustering reads all 9 indicators and sees a wealthy state with bad
    health outcomes. The two methods disagreeing isolates the one country whose need and wealth
    signals point opposite ways

## 8. Wrap-up

- [x] Update `README.md` — rewritten around the project flow: dataset, the four EDA questions
  with their numbers, the preprocessing table, k selection, PCA loadings, the three segments in
  **original units**, the 43-country shortlist, the Q2 cross-check, and a Limitations section.
  The repo-structure block now matches reality (`src/` and `notebooks/` are gone from it)
  - real-unit cluster means were obtained by joining labels back to the raw CSV, since the
    notebook profile is still in transformed space (§7)
- [x] Ten figures exported to `docs/images/` and embedded in the README (~828 KB)
  - produced by running the notebooks' own chart cells under `Agg` and saving instead of
    showing, so they track the committed code. Regenerate after any chart or `k` change
  - not lifted from the stored notebook outputs on purpose: eda's two Q4 charts had no saved
    image at all, and modelling's four were left over from the superseded k=4 run
- [x] Note the environment in the README — `py -3.13`, no venv, no `requirements.txt`, and the
  dependency list. An actual `requirements.txt` is still unwritten if you want one
- [x] Restart-and-run-all: each notebook executes clean from a fresh kernel
  - `modelling.ipynb` — user's own restart before `7139368`, exec 1–13, no error outputs
  - `eda.ipynb` — all code cells run to completion during the figure export, exit 0
  - `preprocessing.ipynb` — exit 0, and **reproduces the committed CSV exactly** (save path
    redirected to scratch, then compared: same shape, same column order, same index order,
    max absolute difference 0.0). The tracked model matrix is genuinely regenerable
- [ ] Final read-through: every chart title states a finding, no clipped labels
  - the ten exported figures were each looked at: nine clean, and the PCA 3D was cropped to its
    content bounds (1210×880 → 968×689) because `subplots_adjust` left ~30% dead margin
  - still open: chart *titles* in `modelling.ipynb` name variables rather than state findings (§4)
- [ ] Commit — `README.md` and `docs/` are untracked/uncommitted as of this check

---

## Conventions to check against before ticking a chart box

- One y-axis per plot; separate panels for separate scales
- `SEQ` for magnitude, `DIV` for signed, `TIER_COLORS` for income tiers
- Log axis for `income` / `gdpp`; `PowerNorm` when colouring by a skewed variable
- Title states the finding, not the variable names
- Chart cell is followed by a markdown cell with the numbers
