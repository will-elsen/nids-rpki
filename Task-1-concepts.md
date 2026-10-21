[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | Task 1 ⮕ | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)

# Task 1 Guidance: RPKI Concepts

This page provides implementation guidance for Task 1 of [nids-rpki.ipynb](nids-rpki.ipynb).

## What this task checks

Task 1 has no code. It checks that the model of RPKI the rest of the assignment depends on is
solid before you implement it: what a ROA states, what ROV does with it, what Max Length
controls, and where origin validation stops. Every answer can be built from the RPKI
background in [Introduction.md](Introduction.md).

---

## Where to look

| Question | Background section in Introduction.md |
| --- | --- |
| Q1 | *Route Origin Authorization (ROA)* and *Route Origin Validation (ROV)* |
| Q2 | the routing-policy list under *Route Origin Validation (ROV)* |
| Q3 | the Max Length discussion under *Route Origin Authorization (ROA)*, and *How RPKI stops Sub-Prefix Hijacking* |
| Q4 | *How RPKI stops Prefix Hijacking* and *What RPKI does not do* |

---

## Reasoning about Max Length

A ROA's Max Length authorizes **every** prefix length from the ROA's own prefix length down
to Max Length, for every sub-block of the prefix — not just the specific announcements the
operator happens to make. Work Q3 in both directions:

- What happens to an announcement **longer** than Max Length, even from the right origin?
- What does a Max Length **longer than needed** quietly authorize that the operator never
  announces, and who could use that?

> **Note:** an operator who announces a covering prefix and one specific sub-prefix has more
> than one way to write ROAs for them. Consider whether a single ROA is the tightest option.

---

## Origin versus path

ROV compares exactly two facts from a route — its prefix and its origin AS — against the
VRPs. For Q4, write out the (prefix, origin) pair each attack presents to a validating
router and compare it with what the ROA authorizes.

---

## What Your Write-Up Should Address

- **Q1** — one sentence defining a ROA and one defining ROV, each naming what it contains or
  checks.
- **Q2** — what Not Found does and does not tell a router, and the reachability cost of
  dropping it given how much of the internet has no ROA.
- **Q3** — the Max Length to use, what breaks at `/24`, and what an over-long Max Length
  would expose.
- **Q4** — which part of the route each attack lies about, and why ROV sees one lie but not
  the other.

[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | Task 1 ⮕ | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)
