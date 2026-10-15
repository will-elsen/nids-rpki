[README](README.md) | [Tasks](Tasks.md) | [RPKI](rpki_understanding.md) | Task 2 ⮕ | [ROV Exercise](rpki_rov_exercise.ipynb) | [MANRS](manrs.md) | [Tasks 4–7](Task-deployment-exercise.md) | [Deployment Exercise](rpki_deployment_exercise.ipynb)

# Task 2 Guidance: Implementing ROV

This page prepares you to implement `classify_rov` required in
[Task 2](Tasks.md), inside [rpki_rov_exercise.ipynb](rpki_rov_exercise.ipynb).

## The three-state rule

For an announced prefix **P** originated by **AS A**, ROV reduces to a three-step lookup
against the VRP table:

```
  1. Find every VRP whose prefix COVERS P.
       none found                                       -> "Not Found"
  2. Among those covering VRPs, is there at least one where
       origin ASN == A  AND  P's length <= max_length?
       yes                                              -> "Valid"
  3.   no (covered, but nothing authorizes A at this
          length)                                       -> "Invalid"
```

The two negative states carry very different meanings. **Not Found** means *no ROA covers
this prefix at all* — the prefix holder simply has not registered in RPKI. **Invalid**
means *a ROA covers it and affirmatively does not authorize this announcement*. Mixing
them up is the most common bug in a first ROV implementation.

## Coverage testing with `ipaddress`

The standard-library `ipaddress` module does the hard part:

```python
net = ipaddress.ip_network("137.110.222.0/24")
roa = ipaddress.ip_network("137.110.0.0/16")

net.subnet_of(roa)   # True  -> the ROA prefix covers this announcement
net.prefixlen        # 24    -> compare this against the VRP's max_length
```

A VRP covers the announcement when the announced network is a subnet of (or equal to) the
VRP prefix. Note that `subnet_of()` raises a `TypeError` when the two networks belong to
different address families, and the snapshot holds both IPv4 and IPv6 VRPs — compare
`.version` first, or filter the table by its `family` column.

## A shape that works

Iterate over the VRP rows (`vrps.itertuples()` is much faster than `iterrows()`), and
track one piece of state: *have I seen any covering VRP at all?*

1. Parse the announced prefix once, before the loop.
2. For each VRP row: skip it if the families differ; skip it if its prefix does not cover
   the announcement.
3. The row covers the announcement — record that fact. If additionally its `asn` equals
   `origin_asn` **and** the announced prefix length is `<= max_length`, return `"Valid"`
   immediately.
4. After the loop: return `"Invalid"` if any covering VRP was seen, else `"Not Found"`.

One authorizing VRP is enough for Valid — but you must check *every* covering VRP before
you may conclude Invalid. A prefix can legitimately be covered by several ROAs at once,
with different origins, different max lengths, or even different trust anchors.

## Worked example

Given a single VRP `137.110.0.0/16, max_length=20, asn=7377`:

| Announcement | Outcome | Reason |
| --- | --- | --- |
| `137.110.0.0/16` from AS7377 | `Valid` | covered, AS matches, 16 ≤ 20 |
| `137.110.0.0/20` from AS7377 | `Valid` | covered, AS matches, 20 ≤ 20 |
| `137.110.0.0/24` from AS7377 | `Invalid` | covered, AS matches, but 24 > 20 |
| `137.110.0.0/16` from AS64500 | `Invalid` | covered, but AS 64500 ≠ 7377 |
| `8.8.8.0/24` from AS15169 | `Not Found` | no VRP covers this prefix |

## Common pitfalls

**Concluding Invalid at the first non-matching covering VRP.** The first covering VRP you
meet may belong to a different origin while a later one authorizes the route. Keep
scanning until you find an authorizing VRP or run out of rows.

**Forgetting the max-length check.** Matching the origin ASN alone is not authorization.
A `/25` announcement under a `/24 max_length=24` ROA from the right origin is still
Invalid — this is exactly the sub-prefix-hijack defense Max Length exists for.

**Returning Not Found when covered.** If any VRP covers the prefix, the answer is Valid
or Invalid — never Not Found. Set a flag the moment a covering VRP is seen.

**Mixed address families.** Calling `subnet_of()` across IPv4/IPv6 raises `TypeError`.
Compare `.version` (or filter on the `family` column) before the coverage test.

**Type mismatches on the ASN.** The `asn` column holds plain integers. If you compare
against a string (`"15169"`) or an `"AS15169"`-style value, everything silently becomes
Invalid or Not Found. Run the hand-built test table first — it isolates each of these
mistakes before you touch 400,000 real rows.

[README](README.md) | [Tasks](Tasks.md) | [RPKI](rpki_understanding.md) | Task 2 ⮕ | [ROV Exercise](rpki_rov_exercise.ipynb) | [MANRS](manrs.md) | [Tasks 4–7](Task-deployment-exercise.md) | [Deployment Exercise](rpki_deployment_exercise.ipynb)
