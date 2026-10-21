[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | Task 3 ⮕ | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)

# Task 3 Guidance: ROV at Internet Scale

This page provides implementation guidance for Task 3 of [nids-rpki.ipynb](nids-rpki.ipynb).

## Why the row scan no longer works

Task 3 classifies ~1.1 million announced (prefix, origin) pairs against ~1 million VRPs. A
scan of the whole VRP table per announcement is ~10¹² comparisons — days of compute for a
decision a router makes millions of times a day. The classification *rule* is unchanged
from Task 2; only step 1 (find the covering VRPs) needs to get fast.

---

## How the provided index works

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

---

## Worked example: masking to a VRP length

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
announcement, so it can never cover it. The lookup only establishes *coverage* — your
classifier still has to check the origin and max length. Here, an announcement at /25 from
AS64500 against `max_length=24` comes out `"Invalid"` (invalid length), and from any other
AS `"Invalid"` (wrong origin) as well.

---

## Building `classify_rov_indexed`

`classify_rov_indexed(prefix, origin_asn, vrp_index)` applies the Task 2 rule to the output
of `covering_vrps()`:

1. An empty covering list settles the answer immediately.
2. Compute the announced prefix length once.
3. Each covering element is a **plain tuple** `(vrp_prefix, asn, max_length)`, not a
   DataFrame row. One element with the right origin and a large enough max length makes the
   announcement Valid.
4. Covered but never authorized is the remaining state.

The states and their priority are identical to Task 2: one authorizing VRP is enough for
Valid, every covering VRP must be checked before concluding Invalid, and *covered but
unauthorized* (Invalid) is a different state from *not covered at all* (Not Found).

---

## Common pitfalls

**Treating the covering list like a DataFrame.** `covering_vrps` returns a list of
tuples. Unpack each element into its three fields — there is no `.asn` attribute or
`["asn"]` column here.

**Re-doing the coverage test.** Everything `covering_vrps` returns already covers the
announcement. Your loop only needs the origin and max-length checks.

> **Gotcha:** `annotate_rov()` caches the *output of your classifier* under `data/`. If you
> change `classify_rov_indexed` after the annotate cell has run, rerun it with
> `force_refresh=True` (or delete the cached file) — otherwise Tasks 4–6 quietly analyze
> your old, buggy states.

**Skipping the hand-built tests.** The 7 test cases each isolate one part of the rule
(wrong origin, beyond max length, second covering VRP, no coverage). Make all of them
pass before spending minutes classifying a million real rows.

---

## What Your Write-Up Should Address

Task 3 has no written questions. Its deliverables are `classify_rov_indexed` passing all 7
hand-built test cases, and `bgp_classified`, the full routing table with a `rov_state`
column, which Tasks 4–6 analyze.

[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | Task 3 ⮕ | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)
