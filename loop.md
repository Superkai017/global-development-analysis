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

## 4. Modelling notebook — scaffolded only

`notebook/modelling.ipynb` is two cells: the imports (`1e89ee29`) and the data load (`40941ec2`).

- [ ] Load with `pd.read_csv(data, index_col='country')` — right now country comes back as a
  plain column, so it lands in the model matrix as a non-numeric feature
- [ ] Swap the absolute `D:\Country-development-analysis\…` path for a relative one, the way
  `preprocessing.ipynb` does (`Path('..') / 'data' / …`) — `eda.ipynb` (`1ff645e7`) has the
  same absolute path and the same fix
- [ ] `StandardScaler` / `PowerTransformer` are imported but the data arrives already scaled —
  drop them unless something actually re-fits
- [ ] No chart tokens in this notebook: `SURFACE`, `SEQ`, `style()`, `label_points()` all live
  in `eda.ipynb` cell `09fffd3d`. First chart cell here needs them copied in or the setup
  cell repeated

## 5. PCA — not started

- [ ] Fit PCA on the scaled matrix
- [ ] Scree / cumulative-variance plot, pick the number of components
- [ ] Read the loadings — what does PC1 actually mean in development terms?
- [ ] Decide: cluster on components or on the scaled features

## 6. Clustering — not started

- [ ] K-Means: elbow (inertia) + silhouette to choose `k`
  - `silhouette_score`, `calinski_harabasz_score` and `davies_bouldin_score` are already
    imported in `1e89ee29` — three metrics, so say which one is deciding
  - for reference: the ad-hoc k=3 run during the preprocessing debugging gave 43 / 72 / 52
    and held across seeds. That was a sanity check, not committed work — it still needs
    doing properly here
- [ ] Hierarchical: dendrogram, pick the cut, compare the grouping to K-Means
- [ ] Lock in a final labelling and attach it to `df`

## 7. Cluster profiling & the actual answer

- [ ] Mean of each of the 9 indicators per cluster
  - centroids are in transformed units — `inverse_transform` back through the scaler and the
    `PowerTransformer` before quoting a number as a percentage or a dollar figure
- [ ] Name the clusters in plain language (e.g. "needs aid" / "developing" / "developed")
- [ ] Scatter of the clusters on the two strongest axes, labelled
- [ ] Produce the aid shortlist — which countries fall in the neediest cluster
- [ ] Sanity check: does the shortlist agree with the Q2 shortlist? Explain any difference

## 8. Wrap-up

- [ ] Update `README.md` — the repo-structure block describes a layout that doesn't exist:
  `data/Country-data.csv` (now `data/raw_data/`), `notebooks/country_clustering.ipynb` (now
  three notebooks under `notebook/`), `src/`, `requirements.txt`. `data/preprocessed_data/`
  isn't mentioned at all and it's tracked
- [ ] Write a `requirements.txt` (or note in the README that it's `py -3.13`, no venv)
- [ ] Restart-and-run-all: each notebook executes clean from a fresh kernel
- [ ] Final read-through: every chart title states a finding, no clipped labels
- [ ] Commit

---

## Conventions to check against before ticking a chart box

- One y-axis per plot; separate panels for separate scales
- `SEQ` for magnitude, `DIV` for signed, `TIER_COLORS` for income tiers
- Log axis for `income` / `gdpp`; `PowerNorm` when colouring by a skewed variable
- Title states the finding, not the variable names
- Chart cell is followed by a markdown cell with the numbers
