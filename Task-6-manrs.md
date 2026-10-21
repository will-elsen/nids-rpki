[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | Task 6 ⮕ | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)

# Task 6 Guidance: MANRS Participants Versus Everyone Else

This page provides implementation guidance for Task 6 of [nids-rpki.ipynb](nids-rpki.ipynb).

## The question being asked

Every ASN in `manrs_asns` belongs to an operator that publicly committed to the MANRS
actions — including publishing ROAs. Everyone else made no such promise. If the
commitment means anything, it should be visible in data the participants don't control.
You will test that three ways: per announcement, per AS, and across the whole
distribution.

---

## Labeling the groups

`Series.isin(a_set)` returns a boolean Series that is `True` where the value is in the set,
and `Index.isin(a_set)` does the same for an index. That one call labels both
`bgp_classified` (by its `origin` column) and `per_as` (by its ASN index).

---

## Task 6.1: announcement-level comparison

`pd.crosstab(rows, columns, normalize=...)` counts every combination of two labels and can
normalize the result. On a toy table:

```python
pets = pd.DataFrame({
    "owner_is_child": [True, True, False, False, False],
    "species":        ["cat", "dog", "dog", "dog", "fish"],
})

pd.crosstab(pets["owner_is_child"], pets["species"], normalize="index") * 100
#                  cat    dog   fish
# False            0.0   66.7   33.3
# True            50.0   50.0    0.0
```

`normalize="index"` makes each **row** sum to 100%, so two groups of very different sizes
become comparable. (`normalize="columns"` would answer a different question — what share of
each column comes from each group — which is dominated by group size.)

---

## Task 6.2: per-AS summary

Group `per_as` by the MANRS label and aggregate one row per group with named aggregations,
as in Task 5. Two of the four columns are not built-in aggregations: `share_zero` is the
share of the group's ASes with `valid_frac == 0`, and `share_full` the share with
`valid_frac == 1`. A named aggregation accepts any function of the group's Series,
including a `lambda`, and the boolean-mean trick from Task 5 turns each comparison into a
share. `"median"` is a built-in aggregation name.

---

## Task 6.3: comparing the two distributions

An empirical CDF plots a group's sorted values against evenly spaced quantiles. For one toy
sample:

```python
sample = pd.Series([0.0, 1.0, 0.5, 1.0]).sort_values()
plt.plot(sample, np.linspace(0, 1, len(sample)), label="sample")
```

Draw one such curve per group on the same axes, with a legend. Read a point on a curve as:
*this fraction of the group's ASes has `valid_frac` at or below x*. A curve that hugs the
bottom until x = 1 is the better-covered group.

---

## Common pitfalls

**Comparing groups by raw counts.** The non-MANRS group has dozens of times more ASes.
Every comparison must be normalized within the group — that is what
`normalize="index"`, medians, and shares are for.

**Invisible participants.** Only ASes that actually originate prefixes appear in `bgp` —
a MANRS ASN that announces nothing is absent from both tables. Your `n_ases` for the
MANRS group will therefore be smaller than `len(manrs_asns)`; that is expected, not a
bug.

**Stale `per_as`.** Tasks 6.2 and 6.3 reuse the Task 5 DataFrame. If you rebuilt
`bgp_classified` (for example after a `force_refresh`), rebuild `per_as` too before
adding the `manrs` column.

> **Note:** this is an observational comparison, not an experiment. The MANRS list is also
> fetched live, while the routing table and VRPs are pinned to `SNAPSHOT_DATE`; Task 7 returns
> to that mismatch.

---

## What Your Write-Up Should Address

- **Q11** — the gap between the groups on `median_valid_frac` and `share_zero`, one causal
  mechanism in each direction (participation causing coverage, and self-selection), and the
  data that would tell them apart.

[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | Task 6 ⮕ | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)
