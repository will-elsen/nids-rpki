[README](README.md) | [Tasks](Tasks.md) | [RPKI](rpki_understanding.md) | [Task 2](Task-rov.md) | [ROV Exercise](rpki_rov_exercise.ipynb) | [MANRS](manrs.md) | Tasks 4–7 ⮕ | [Deployment Exercise](rpki_deployment_exercise.ipynb)

# Tasks 4–7 Guidance: The Deployment Exercise

This page prepares you for the four coding tasks in
[rpki_deployment_exercise.ipynb](rpki_deployment_exercise.ipynb): the index-based ROV
classifier (Task 4) and the three analyses built on top of it (Tasks 5–7). The full task
list and questions are in [Tasks](Tasks.md).

---

## Task 4: ROV at Internet Scale

### Why the row scan no longer works

The deployment notebook classifies ~1.1 million announced (prefix, origin) pairs against
~1 million VRPs. A scan of the whole VRP table per announcement is ~10¹² comparisons —
days of compute for a decision a router makes millions of times a day. The classification
*rule* is unchanged from the ROV exercise; only step 1 (find the covering VRPs) needs to
get fast.

### How the provided index works

The key observation: a VRP prefix can only cover an announced prefix if it **equals the
announcement's network address masked to the VRP's length**. So instead of scanning,
mask the announced address to each prefix length that occurs in the VRP table (about 25
of them) and look each candidate up in a dictionary — ~25 hash lookups per announcement
instead of a million-row scan. Real validators and routers use a radix trie for the same
job; this is the flat-dictionary version of that idea.

The notebook provides both halves — read them before building on top:

| Provided function | What it does |
| --- | --- |
| `build_vrp_index(vrps_df)` | returns `(table, lengths)`: `table` maps `(network_int, length)` → list of `(vrp_prefix, asn, max_length)` tuples; `lengths` is the sorted list of prefix lengths present |
| `covering_vrps(prefix, vrp_index)` | returns every VRP tuple whose prefix covers `prefix` (IPv4 only) |

### Worked example: masking to a VRP length

Masking an address to length *L* keeps its first *L* bits and zeroes the rest — which is
exactly "the network address a VRP of length *L* would have to carry to cover this
announcement." In code (this is what `covering_vrps` does):

```python
mask = (0xFFFFFFFF << (32 - vrp_length)) & 0xFFFFFFFF
candidate = address & mask     # both as 32-bit integers
```

Take the announcement `203.0.113.128/25`, and suppose the VRP table holds one entry,
`203.0.113.0/24 (asn=64500, max_length=24)`, so the lengths present are `[16, 24]`.
Masking `203.0.113.128` to /24 in binary (last two octets shown):

```
address          203 . 0 . 01110001 . 10000000     (203.0.113.128)
/24 mask         255 . 255 . 11111111 . 00000000   (0xFFFFFF00)
address & mask   203 . 0 . 01110001 . 00000000     (203.0.113.0)
```

The trailing `10000000` host bits are zeroed; the result is the network address the
announcement would fall under at /24. Each candidate length gets one dictionary lookup:

| VRP length | mask | masked address | lookup key | in the table? |
| --- | --- | --- | --- | --- |
| 16 | `255.255.0.0` | `203.0.0.0` | `(203.0.0.0, 16)` | no — nothing registered there |
| 24 | `255.255.255.0` | `203.0.113.0` | `(203.0.113.0, 24)` | **yes** — `203.0.113.0/24` covers the announcement |

Only lengths ≤ 25 are tried: a /26 VRP occupies a smaller block than the /25
announcement, so it can never cover it. Note the lookup only establishes *coverage* —
your classifier still has to check the origin and max length. Here, an announcement at
/25 from AS64500 against `max_length=24` comes out `"Invalid"` (invalid length), and from
any other AS `"Invalid"` (wrong origin) as well.

### Your job

Fill in `classify_rov(prefix, origin_asn, vrp_index)`:

1. `covering = covering_vrps(prefix, vrp_index)` — if empty, return `"Not Found"`.
2. Compute the announced prefix length once.
3. Loop over `covering`. Each element is a **plain tuple** `(vrp_prefix, asn,
   max_length)`, not a DataFrame row. If `asn == origin_asn` and the announced length is
   `<= max_length`, return `"Valid"` immediately.
4. Fell through the loop → covered but unauthorized → return `"Invalid"`.

The states and their priority are identical to Task 2: one authorizing VRP is enough for
Valid, every covering VRP must be checked before concluding Invalid, and *covered but
unauthorized* (Invalid) is a different state from *not covered at all* (Not Found).

### Common pitfalls

**Treating the covering list like a DataFrame.** `covering_vrps` returns a list of
tuples. Unpack with `for vrp_prefix, asn, max_length in covering:` — there is no `.asn`
attribute or `["asn"]` column here.

**Re-doing the coverage test.** Everything `covering_vrps` returns already covers the
announcement. Your loop only needs the origin and max-length checks.

**Stale cache after fixing your classifier.** `annotate_rov()` caches the *output of your
classifier* under `deployment_cache/`. If you change `classify_rov` after the annotate
cell has run, rerun it with `force_refresh=True` (or delete the cached file) — otherwise
Tasks 5–7 quietly analyze your old, buggy states.

**Skipping the hand-built tests.** The 7 test cases each isolate one part of the rule
(wrong origin, beyond max length, second covering VRP, no coverage). Make all of them
pass before spending minutes classifying a million real rows.

---

## Task 5: Internet-Wide ROV Shares

### What to produce

From `bgp_classified` (the full routing table with your `rov_state` column):

1. a table with one row per ROV state — **count** of announcements and **percentage
   share** of announcements in that state
2. a **bar chart** of the percentage shares

### The tools

`value_counts()` does almost all of the work. Called plainly it gives the counts; with
`normalize=True` it gives fractions, which `* 100` turns into percentages:

```python
counts = bgp_classified["rov_state"].value_counts()
shares = bgp_classified["rov_state"].value_counts(normalize=True) * 100
```

Both Series share the same index (the three state names), so they combine into one table
in a single call:

```python
summary = pd.DataFrame({"count": counts, "percent": shares.round(2)})
```

For the chart, `Series.plot.bar()` gives you the figure in one line. Label the y-axis as
a percentage and give the figure a title — a chart with three unlabeled bars answers
nothing.

### Reading the result

The unit here is the **(prefix, origin) pair** — not addresses, not ASes. A `/8` and a
`/24` each count once, and a prefix announced by two origins counts twice. Keep that in
mind when you word your answer to Q1.

If your code is correct, Valid and Not Found should together dominate the table, with
Invalid a small single-digit-percentage share. If Invalid comes out large, or Not Found
is nearly zero, revisit your Task 4 classifier (and remember the `annotate_rov` cache —
`force_refresh=True` after any fix).

### Common pitfalls

**Reporting fractions as percentages.** `normalize=True` returns values in `[0, 1]`.
Multiply by 100 (or label the axis as a fraction) — a bar chart claiming 0.5% Valid when
you meant 50% is a very different internet.

**Dropping a state.** All three states should appear in the table even if you reorder
them. `value_counts()` sorts by frequency; `reindex(["Valid", "Invalid", "Not Found"])`
puts them in a fixed, comparable order.

---

## Task 6: Per-AS ROA Coverage

### Changing the unit of analysis

Task 5 counted announcements; this task counts **networks**. The internet-wide number
blends an AS announcing thousands of prefixes with an AS announcing two, but publishing
ROAs is a per-operator decision — so the telling question is: for each origin AS, what
fraction of *its own* announcements does RPKI vouch for?

### What to produce

A DataFrame `per_as`, **indexed by origin ASN**, with exactly these columns (Task 7
reuses this DataFrame and these names):

| column | meaning |
| --- | --- |
| `n_announcements` | how many (prefix, origin) pairs the AS originates |
| `valid_frac` | the fraction of them classified **Valid** |

Then a histogram of `valid_frac` across all origin ASes — 20 bins works well.

### The boolean-mean trick

The mean of a boolean column is the fraction of `True` values. That makes the whole task
one groupby:

```python
bgp_classified["is_valid"] = bgp_classified["rov_state"] == "Valid"

per_as = bgp_classified.groupby("origin").agg(
    n_announcements=("rov_state", "size"),
    valid_frac=("is_valid", "mean"),
)
```

`groupby("origin")` leaves the origin ASN as the index, which is exactly what Task 7's
`per_as.index.isin(manrs_asns)` expects — don't `reset_index()`.

For the plot, `per_as["valid_frac"].plot.hist(bins=20)` draws the histogram; label the
x-axis (fraction of the AS's announcements that are Valid) and the y-axis (number of
ASes).

### Reading the result

Expect a strongly **bimodal** shape: a spike at 0 (the AS published nothing usable) and a
spike at 1 (every announcement Valid), with comparatively little in between. Q2 asks you
to explain why the operational decision behind ROA publishing produces exactly this
all-or-nothing distribution, and what kind of AS lands in the middle.

### Common pitfalls

**Counting the wrong states.** `valid_frac` is the share of announcements that are
**Valid** — not "Valid or Not Found". An AS whose announcements are all Not Found has
`valid_frac == 0`.

**Renaming or re-indexing.** Task 7 breaks if the index is not the origin ASN or the
columns are not `n_announcements` / `valid_frac`. Keep the shape exactly as specified.

**Weighting by announcements in the histogram.** The histogram counts **ASes**, one unit
each, regardless of how many prefixes they announce. That is the point of the task — do
not weight bars by `n_announcements`.

---

## Task 7: The MANRS Comparison

### The question being asked

Every ASN in `manrs_asns` belongs to an operator that publicly committed to the MANRS
actions — including publishing ROAs. Everyone else made no such promise. If the
commitment means anything, it should be visible in data the participants don't control.
You will test that three ways.

### Deliverable 1: announcement-level comparison

Label every row of `bgp_classified` with whether its origin is a MANRS ASN, then show the
percentage of announcements in each ROV state for the two groups:

```python
bgp_classified["manrs"] = bgp_classified["origin"].isin(manrs_asns)

pd.crosstab(bgp_classified["manrs"], bgp_classified["rov_state"],
            normalize="index") * 100
```

`normalize="index"` makes each **row** sum to 100%, so the two groups are comparable even
though one is vastly larger than the other. (`normalize="columns"` would answer a
different question — what share of Valid routes comes from MANRS members — which is
dominated by group size.)

### Deliverable 2: per-AS summary

Add the group label to the `per_as` DataFrame from Task 6, then aggregate one row per
group. The boolean-mean trick from Task 6 works for the two share columns:

```python
per_as["manrs"] = per_as.index.isin(manrs_asns)

summary = per_as.groupby("manrs").agg(
    n_ases=("valid_frac", "size"),
    median_valid_frac=("valid_frac", "median"),
    share_zero=("valid_frac", lambda s: (s == 0).mean()),
    share_full=("valid_frac", lambda s: (s == 1).mean()),
)
```

`share_zero` is the share of the group's ASes that published nothing usable;
`share_full` the share whose every announcement is Valid.

### Deliverable 3: comparing the two distributions

A pair of empirical CDF curves shows the whole `valid_frac` distribution for each group
in one picture. For each group, sort the values and plot them against evenly spaced
quantiles:

```python
for label, group in per_as.groupby("manrs"):
    values = group["valid_frac"].sort_values()
    plt.plot(values, np.linspace(0, 1, len(values)),
             label="MANRS" if label else "non-MANRS")
plt.xlabel("valid_frac"); plt.ylabel("fraction of ASes ≤ x"); plt.legend()
```

Read a point on a curve as: *this fraction of the group's ASes has `valid_frac` at or
below x*. A curve that hugs the bottom until x = 1 is the better-covered group.

### Common pitfalls

**Comparing groups by raw counts.** The non-MANRS group has hundreds of times more ASes.
Every comparison must be normalized within the group — that is what
`normalize="index"`, medians, and shares are for.

**Invisible participants.** Only ASes that actually originate prefixes appear in `bgp` —
a MANRS ASN that announces nothing is absent from both tables. Your `n_ases` for the
MANRS group will therefore be smaller than `len(manrs_asns)`; that is expected, not a
bug.

**Stale `per_as`.** Deliverables 2 and 3 reuse the Task 6 DataFrame. If you rebuilt
`bgp_classified` (for example after a `force_refresh`), rebuild `per_as` too before
adding the `manrs` column.

**Over-reading the result.** This is an observational comparison, not an experiment — Q3
explicitly asks for one causal mechanism in each direction and what data would separate
them. Keep the timing mismatch in mind as well: the MANRS list is fetched live, while the
routing table and VRPs are pinned to `SNAPSHOT_DATE` (the final reflection returns to
this).

[README](README.md) | [Tasks](Tasks.md) | [RPKI](rpki_understanding.md) | [Task 2](Task-rov.md) | [ROV Exercise](rpki_rov_exercise.ipynb) | [MANRS](manrs.md) | Tasks 4–7 ⮕ | [Deployment Exercise](rpki_deployment_exercise.ipynb)
