# EDA Checklist — Python & Pandas

A step-by-step reference for exploratory data analysis, based on a structured five-phase workflow. Follow these phases in order — skipping ahead tends to waste effort on columns or rows you'll later discard.

---

## Phase 1 — Setup & Imports

- [ ] Import core libraries: `pandas`, `numpy`
- [ ] Import viz libraries: `matplotlib.pyplot`, `seaborn`
- [ ] Set plot style: `plt.style.use('ggplot')`
- [ ] Expand column display: `pd.set_option('display.max_columns', 200)`
- [ ] Load data: `pd.read_csv()`

---

## Phase 2 — Data Understanding

- [ ] Check shape: `df.shape` — rows × columns
- [ ] Preview rows: `df.head()` / `df.tail()`
- [ ] List all columns: `df.columns`
- [ ] Inspect data types: `df.dtypes`
- [ ] Summary statistics: `df.describe()` for numeric columns

---

## Phase 3 — Data Preparation

- [ ] Subset columns — select only relevant columns; reassign with `.copy()`
- [ ] Alternatively, drop unwanted columns: `df.drop(cols, axis=1)`
- [ ] Fix data types — cast dates with `pd.to_datetime()`, numerics with `pd.to_numeric()`
- [ ] Rename columns for clarity: `df.rename(columns={old: new})`
- [ ] Identify missing values: `df.isna().sum()` per column
- [ ] Find exact duplicates: `df.duplicated()`
- [ ] Find key-level duplicates: `df.duplicated(subset=[...])`
- [ ] Inspect a specific duplicate with `df.query()` to understand why
- [ ] Remove duplicates using `~df.duplicated(subset=[...])` and `.reset_index(drop=True)`

---

## Phase 4 — Feature Understanding (Univariate)

- [ ] Count categorical values: `df[col].value_counts()`
- [ ] Plot top categories as a bar chart: `.value_counts().head(n).plot(kind='bar')`
- [ ] Plot numeric distribution as histogram: `.plot(kind='hist', bins=20)`
- [ ] Overlay a KDE (density) curve: `.plot(kind='kde')`
- [ ] Add axis labels and title to every plot

---

## Phase 5 — Feature Relationships (Multivariate)

- [ ] Scatter plot two features: `df.plot(kind='scatter', x=..., y=...)`
- [ ] Enhance scatter with a third variable using Seaborn's `hue` parameter
- [ ] All-pairs overview: `sns.pairplot(df, vars=[...], hue=...)`
- [ ] Compute correlation matrix: `df[num_cols].dropna().corr()`
- [ ] Visualise correlations: `sns.heatmap(corr, annot=True)`

---

## Phase 6 — Analytical Question

- [ ] State a specific, answerable question about the data
- [ ] Filter irrelevant rows: `df.query()` to exclude noise categories
- [ ] Group and aggregate: `df.groupby(key)[col].agg(['mean', 'count'])`
- [ ] Apply secondary filters (e.g. minimum group size) on the aggregated result
- [ ] Sort results: `.sort_values(by=...)`
- [ ] Visualise the answer as a horizontal bar chart with labeled axes

---

## Key Highlights

### Always `.copy()` when subsetting

```python
df = df[keep_cols].copy()
```

Reassigning a slice without `.copy()` creates a *view*, not a new object. You'll hit `SettingWithCopyWarning` the moment you mutate anything downstream — the fix is always to add `.copy()` at the point of subsetting, not later.

---

### KDE over histogram for comparisons

```python
df['Speed_mph'].plot(kind='kde')
```

Histograms are fine for a single variable. KDE normalises the y-axis so overlaid distributions are directly comparable — much more useful when you need to compare subgroups (e.g. steel vs wood coasters, fraud vs non-fraud).

---

### The cleanest deduplication pattern

```python
mask = ~df.duplicated(subset=['Name', 'Location', 'Opening_date'])
df = df.loc[mask].reset_index(drop=True)
```

The tilde (`~`) inverts the boolean mask, keeping the *first* occurrence and dropping all subsequent ones. This is less error-prone than `drop_duplicates(keep='first')` because you can inspect the mask before applying it — just `df.loc[~mask]` to see exactly what will be dropped.

---

### Prepare before you explore

Clean column selection and deduplication belong in Phase 3, *before* any univariate or multivariate analysis. Spending time plotting a column you'll later drop — or on a distribution skewed by duplicates — is wasted effort. This ordering is deliberate.

---

### The reusable "top N by metric" query pattern

```python
result = (
    df.query("Location != 'Other'")
      .groupby('Location')['Speed_mph']
      .agg(['mean', 'count'])
      .query("count >= 10")
      .sort_values('mean', ascending=False)
)
result['mean'].plot(kind='barh', title='Avg coaster speed by location')
plt.xlabel('Average speed (mph)')
plt.show()
```

This `groupby → agg → filter on count → sort` chain answers almost any *"top X by metric with a minimum sample size"* question. The minimum count filter is important: a park with one very fast coaster would otherwise dominate the ranking.

---

### Use `plt.show()` for clean notebook output

Without `plt.show()`, Pandas/Matplotlib prints the axes object representation alongside the plot. Adding `plt.show()` at the end of a plot cell suppresses the text output and keeps the notebook readable.

---

*Source: Exploratory Data Analysis with Python & Pandas — Rob Mulla (Kaggle Grandmaster)*
