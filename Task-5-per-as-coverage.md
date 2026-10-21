[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | Task 5 ⮕ | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)

# Task 5 Guidance: Per-AS ROA Coverage

This page provides implementation guidance for Task 5 of [nids-rpki.ipynb](nids-rpki.ipynb).

## Changing the unit of analysis

Task 4 counted announcements; this task counts **networks**. The internet-wide number
blends an AS announcing thousands of prefixes with an AS announcing two, but publishing
ROAs is a per-operator decision — so the telling question is: for each origin AS, what
fraction of *its own* announcements does RPKI vouch for?

---

## What to produce

A DataFrame `per_as`, **indexed by origin ASN**, with exactly these columns (Task 6
reuses this DataFrame and these names):

| column | meaning |
| --- | --- |
| `n_announcements` | how many (prefix, origin) pairs the AS originates |
| `valid_frac` | the fraction of them classified **Valid** |

Then a histogram of `valid_frac` across all origin ASes — 20 bins works well.

---

## The boolean-mean trick

The mean of a boolean column is the fraction of `True` values, which turns "what fraction of
each group's rows satisfy X" into a single `groupby`. On a toy table of orders:

```python
orders = pd.DataFrame({
    "store":   ["a", "a", "a", "b", "b"],
    "on_time": [True, True, False, True, True],
})

orders.groupby("store").agg(
    n_orders=("on_time", "size"),       # rows per store
    on_time_frac=("on_time", "mean"),   # a: 0.667, b: 1.0
)
```

`groupby(...)` leaves the grouping key as the index, which is exactly the shape Task 6
expects — don't `reset_index()`. For the plot, `Series.plot.hist(bins=20)` draws the
histogram; label the x-axis (fraction of the AS's announcements that are Valid) and the
y-axis (number of ASes).

---

## Reading the result

Expect a strongly **bimodal** shape: a spike at 0 (the AS published nothing usable) and a
spike at 1 (every announcement Valid), with comparatively little in between. Q10 asks you
to explain why the operational decision behind ROA publishing produces exactly this
all-or-nothing distribution, and what kind of AS lands in the middle.

---

## Common pitfalls

**Counting the wrong states.** `valid_frac` is the share of announcements that are
**Valid** — not "Valid or Not Found". An AS whose announcements are all Not Found has
`valid_frac == 0`.

**Renaming or re-indexing.** Task 6 breaks if the index is not the origin ASN or the
columns are not `n_announcements` / `valid_frac`. Keep the shape exactly as specified.

**Weighting by announcements in the histogram.** The histogram counts **ASes**, one unit
each, regardless of how many prefixes they announce. That is the point of the task — do
not weight bars by `n_announcements`.

---

## What Your Write-Up Should Address

- **Q10** — the shape of the `valid_frac` histogram, why ROA publishing as an operational
  decision produces it, and what kind of AS sits in the middle.

[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | Task 5 ⮕ | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)
