[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | Task 7 ⮕ | [Notebook](nids-rpki.ipynb)

# Task 7 Guidance: The BGPsec and ASPA Footprint

This page provides implementation guidance for Task 7 of [nids-rpki.ipynb](nids-rpki.ipynb).

## What the provided cell reports

The code for Task 7 is provided. It reads `snapshot`, the live rpki-client summary loaded in
Task 0, and prints four counts: ROAs, the VRPs they expand to, ASPA records, and BGPsec router
certificates, plus the ASNs that hold a router certificate. Your work is interpreting them.
The background on each artifact is in [Introduction.md](Introduction.md), under *Task 7
Background*.

---

## Comparing artifacts fairly

Compare like with like. A ROA, an ASPA record, and a router certificate are each the first
thing a network must **publish** to take part in one mechanism, so their counts are
comparable measures of willingness to start. For each one, ask:

- Can a network publish it **alone**, or does it only do something once its neighbors act?
- What does the publisher **get back**, and when — the day it publishes, or once most paths
  are covered?
- What else must change for it to have effect: router software, BGP messages, crypto load?

> **Note:** the rpki-client snapshot is validated minutes before you download it, so these
> counts describe today, not `SNAPSHOT_DATE`.

---

## Reflecting on what you measured

Q13 and Q14 look back over the whole assignment.

- **Timing.** List each dataset with the date it describes. For Q13, take each scenario in
  turn, decide which group the network is counted in and with which coverage, and follow the
  effect on the gap you reported in Task 6.
- **Visibility.** Everything you measured is something networks *publish*: ROAs, and routes
  seen by collectors. For Q14, ask what trace a network leaves when it *drops* a route, and
  what experiment would make that trace visible from outside.

---

## What Your Write-Up Should Address

- **Q12** — the ROA and router-certificate counts side by side, what a network gets for
  publishing each, and what the ASPA count implies about which path-security designs can
  deploy.
- **Q13** — the direction each timing scenario pushes the MANRS comparison.
- **Q14** — why ROV enforcement is invisible in these datasets, and a concrete design for
  measuring it from the outside.

[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | Task 7 ⮕ | [Notebook](nids-rpki.ipynb)
