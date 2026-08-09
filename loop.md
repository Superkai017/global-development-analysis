# loop.md — project checklist

Working checklist for the country-clustering project. Tick a box when the work is in
`notebook/eda.ipynb`, the chart has been *looked at*, and the cell runs top-to-bottom clean.

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
  - [ ] Decide how "combined" is defined (rank-sum? filter then sort? percentile score?)
  - [ ] Result shape: 15 rows, country + both metrics, ordered by the combined criterion
  - [ ] Chart or table — whichever actually communicates the shortlist
  - [ ] Markdown cell with the finding
- [ ] **Q3 (visualization)** — `income` vs `life_expec`, coloured by `child_mort`
  - [ ] Log x-axis for `income`; `PowerNorm` on the colour scale (`child_mort` is skewed)
  - [ ] Say whether it's linear or plateaus, and where the bend is
  - [ ] Markdown cell with the finding
- [ ] **Q4 (visualization)** — `total_fer` vs `child_mort` + correlation heatmap of all 9 features
  - [ ] Scatter with `label_points()` for the outlier countries
  - [ ] Heatmap on `DIV`, neutral grey at zero
  - [ ] Markdown cell naming which variables carry the "need" signal
- [ ] Any new question added to `question.txt` gets its own block here

## 3. Preprocessing — not started

- [ ] Decide what to do about right-skew in `income`, `gdpp` (and `inflation`) — log-transform or leave
- [ ] Outlier check: Nigeria/Haiti at the high-`child_mort` end, Luxembourg/Qatar at the high-`income` end — keep or cap, and justify it
- [ ] Choose the scaler (`StandardScaler` vs `RobustScaler`) and write down why
- [ ] Confirm the feature matrix: 9 numeric columns, `country` held aside as the index/label

## 4. PCA — not started

- [ ] Fit PCA on the scaled matrix
- [ ] Scree / cumulative-variance plot, pick the number of components
- [ ] Read the loadings — what does PC1 actually mean in development terms?
- [ ] Decide: cluster on components or on the scaled features

## 5. Clustering — not started

- [ ] K-Means: elbow (inertia) + silhouette to choose `k`
- [ ] Hierarchical: dendrogram, pick the cut, compare the grouping to K-Means
- [ ] Lock in a final labelling and attach it to `df`

## 6. Cluster profiling & the actual answer

- [ ] Mean of each of the 9 indicators per cluster
- [ ] Name the clusters in plain language (e.g. "needs aid" / "developing" / "developed")
- [ ] Scatter of the clusters on the two strongest axes, labelled
- [ ] Produce the aid shortlist — which countries fall in the neediest cluster
- [ ] Sanity check: does the shortlist agree with the Q2 shortlist? Explain any difference

## 7. Wrap-up

- [ ] Update `README.md` — repo structure currently describes files that don't exist
  (`notebooks/country_clustering.ipynb`, `src/`, `requirements.txt`)
- [ ] Write a `requirements.txt` (or note in the README that it's `py -3.13`, no venv)
- [ ] Restart-and-run-all: notebook executes clean from a fresh kernel
- [ ] Final read-through: every chart title states a finding, no clipped labels
- [ ] Commit

---

## Conventions to check against before ticking a chart box

- One y-axis per plot; separate panels for separate scales
- `SEQ` for magnitude, `DIV` for signed, `TIER_COLORS` for income tiers
- Log axis for `income` / `gdpp`; `PowerNorm` when colouring by a skewed variable
- Title states the finding, not the variable names
- Chart cell is followed by a markdown cell with the numbers