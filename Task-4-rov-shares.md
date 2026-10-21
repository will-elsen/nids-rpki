[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | Task 4 ⮕ | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)

# Task 4 Guidance: Internet-Wide ROV Shares

This page provides implementation guidance for Task 4 of [nids-rpki.ipynb](nids-rpki.ipynb).

## What to produce

From `bgp_classified` (the full routing table with your `rov_state` column):

1. `rov_summary`, a table with one row per ROV state — the **count** of announcements and the
   **percentage share** of announcements in that state
2. a **bar chart** of the percentage shares

---

## The tools

`value_counts()` does almost all of the work. Called plainly it gives counts; with
`normalize=True` it gives fractions, which `* 100` turns into percentages. On a toy Series:

```python
colors = pd.Series(["red", "red", "blue", "green", "red"])

colors.value_counts()                      # red 3, blue 1, green 1
colors.value_counts(normalize=True) * 100  # red 60.0, blue 20.0, green 20.0
```

Two Series that share an index (here, the color names) combine into one table with
`pd.DataFrame({"name_a": series_a, "name_b": series_b})`, matched row by row on that index.
`Series.plot.bar()` draws the chart in one line; label the y-axis as a percentage and give the
figure a title — a chart with three unlabeled bars answers nothing.

---

## Reading the result

The unit here is the **(prefix, origin) pair** — not addresses, not ASes. A `/8` and a `/24`
each count once, and a prefix announced by two origins counts twice. Keep that in mind when
you word your answer to Q9.

If your code is correct, Valid and Not Found should together dominate the table, with
Invalid a small single-digit-percentage share. If Invalid comes out large, or Not Found is
nearly zero, revisit your Task 3 classifier (and remember the `annotate_rov` cache —
`force_refresh=True` after any fix).

---

## Common pitfalls

**Reporting fractions as percentages.** `normalize=True` returns values in `[0, 1]`.
Multiply by 100 (or label the axis as a fraction) — a bar chart claiming 0.5% Valid when
you meant 50% is a very different internet.

**Dropping or shuffling states.** All three states should appear in the table.
`value_counts()` sorts by frequency; `reindex(["Valid", "Invalid", "Not Found"])` puts them
in a fixed, comparable order.

---

## What Your Write-Up Should Address

- **Q9** — the Valid, Not Found, and Invalid shares of announced (prefix, origin) pairs; who
  is affected by the Invalid share and why it is so small; and what the Not Found share means
  for an operator tempted to drop Not Found routes.

[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | Task 4 ⮕ | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)
