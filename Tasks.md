[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | Tasks ⮕ | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)

# Tasks

Complete the tasks below in order. All tasks are performed inside [nids-rpki.ipynb](nids-rpki.ipynb) — replace the `# YOUR CODE HERE` sections with your code and answer the questions in the markdown cells that follow each code cell.

---

## Task 0: Setup and Data Access

All work is done on **NRP's JupyterHub**. The notebook downloads its datasets directly from public archives and caches them under `data/` (see [Datasets.md](Datasets.md)).

- step 1. Access NRP's JupyterHub: [https://jupyterhub-west.nrp-nautilus.io](https://jupyterhub-west.nrp-nautilus.io)
  - Detailed access instructions: [How to access NRP](https://www.caida.org/projects/nids/how-to/nrp-access/)
  - When spawning your server, choose an instance with **at least 4 GB of memory**; the internet-scale tasks hold about a million VRPs and 1.1 million announcements at once
- step 2. Upload `nids-rpki.ipynb` and run every cell in its **Task 0** section, starting with the setup cell that installs dependencies
- [ ] Verify that the 2023-03-01 VRP snapshot loads (about 400,000 VRPs across five trust anchors)
- [ ] Verify that the 2026-07-01 VRP snapshot loads (close to a million VRPs)
- [ ] Verify that the 2026-07-01 routing table loads (roughly 1.1 million announced (prefix, origin) pairs)
- [ ] Verify that the MANRS participant list and the live rpki-client summary load

---

## Task 1: RPKI Concepts

Detailed guidance: [Task 1 — Concepts](Task-1-concepts.md)

### Task 1.1 — ROAs and ROV

Read the RPKI background in [Introduction.md](Introduction.md), then answer from it.

- [ ] **Q1** In one sentence each, what is a **ROA** and what is **ROV**?
- [ ] **Q2** A router runs ROV on a route and gets **Not Found**. Why is dropping every Not Found route a bad idea today?

### Task 1.2 — Max Length

Work through what the ROA's Max Length field authorizes and what it forbids.

- [ ] **Q3** An operator announces `203.0.113.0/24` and also, sometimes, `203.0.113.0/26`. What should the **Max Length** on their ROA be, and what would break if they set it to `/24`?

### Task 1.3 — What origin validation cannot see

Contrast the two attacks: one RPKI defeats, one it does not.

- [ ] **Q4** Why can a **prefix hijack** be stopped by RPKI, while a **forged-AS_PATH** attack that ends in the real origin can slip through as Valid?

---

## Task 2: Route Origin Validation

Detailed guidance: [Task 2 — Route Origin Validation](Task-2-rov.md)

### Task 2.1 — Implement `classify_rov`

Fill in `classify_rov(announced_prefix, origin_asn, vrps)` so it classifies one announcement against a DataFrame of VRPs and returns `"Valid"`, `"Invalid"`, or `"Not Found"`. A plain scan over the rows is fine. All 9 hand-built test cases in the cell that follows must pass.

### Task 2.2 — Classify real announcements

Run the provided cells that classify the 11 real announcements in `TARGETS` against the 2023-03-01 snapshot (`rov_vrps`, about 400,000 VRPs).

- [ ] **Q5** Three of the targets announce the same address space from the same origin AS at three different prefix lengths (`/18`, `/24`, `/25`). Did all three get the same state? Explain the boundary in terms of the covering ROA's max length.
- [ ] **Q6** Two targets came back **Not Found**. What would a router configured with standard policy do with them, and why is accepting them still the safe choice today?

### Task 2.3 — Diagnose an Invalid announcement

Build `covering`, the rows of `rov_vrps` whose prefix covers one announcement that came back Invalid.

- [ ] **Q7** Pick an announcement that came back **Invalid**. Using the covering VRPs you listed, explain exactly which condition it failed — wrong origin, excessive length, or both.

### Task 2.4 — Trust anchors

Consider what the `trust_anchor` column does and does not mean.

- [ ] **Q8** Your `classify_rov` ignores the `trust_anchor` column entirely. Why is that correct? What would go wrong if you had filtered the VRPs by trust anchor first?

---

## Task 3: ROV at Internet Scale

Detailed guidance: [Task 3 — Internet Scale](Task-3-internet-scale.md)

### Task 3.1 — Index the VRPs

Run the provided `build_vrp_index()` and `covering_vrps()` cells, which index the 2026-07-01 IPv4 VRPs by exact prefix.

### Task 3.2 — Implement `classify_rov_indexed`

Fill in `classify_rov_indexed(prefix, origin_asn, vrp_index)` on top of `covering_vrps()`. The three-state rule is identical to Task 2; only the lookup changes. All 7 hand-built test cases in the cell that follows must pass.

### Task 3.3 — Classify the whole routing table

Run the provided `annotate_rov()` cell. It applies your classifier to about 1.1 million announced (prefix, origin) pairs and produces `bgp_classified` (a minute or two on the first run, cached in `data/` afterwards).

---

## Task 4: How Much of the Internet Implements RPKI?

Detailed guidance: [Task 4 — ROV Shares](Task-4-rov-shares.md)

### Task 4.1 — Internet-wide ROV state shares

From `bgp_classified`, build `rov_summary`: one row per ROV state with the count and percentage share of announcements in that state. Then plot a bar chart of the percentage shares.

- [ ] **Q9** Report your headline numbers: what share of announced (prefix, origin) pairs is **Valid**, **Not Found**, and **Invalid**? Operators who enforce ROV drop Invalid routes — so who is actually affected by the Invalid share you measured, and why is that share so much smaller than the other two? What does the Not Found share say about how much of the internet would go dark if operators dropped Not Found routes today?

---

## Task 5: ROA Coverage, One Network at a Time

Detailed guidance: [Task 5 — Per-AS Coverage](Task-5-per-as-coverage.md)

### Task 5.1 — Per-AS ROA coverage

Build `per_as`, indexed by origin ASN, with columns `n_announcements` and `valid_frac`. Then plot a histogram of `valid_frac` across all origin ASes. Task 6 reuses `per_as`, so keep the index and column names exactly as specified.

- [ ] **Q10** Describe the shape of the histogram. Most ASes sit at one of the two extremes — all or nothing — rather than in the middle. Why does ROA publishing, as an operational decision, produce exactly this shape? What kind of AS would you expect to find *in* the middle?

---

## Task 6: MANRS Participants Versus Everyone Else

Detailed guidance: [Task 6 — MANRS](Task-6-manrs.md)

### Task 6.1 — Announcement-level comparison

Label every row of `bgp_classified` with whether its origin is a MANRS ASN, then build `manrs_crosstab`: the percentage of announcements in each ROV state for the two groups.

### Task 6.2 — Per-AS summary

Label `per_as` the same way, then build `manrs_summary`: one row per group with `n_ases`, `median_valid_frac`, `share_zero`, and `share_full`.

### Task 6.3 — Comparing the distributions

Plot the two groups' `valid_frac` distributions on one figure (two empirical CDF curves work well).

- [ ] **Q11** Quantify the gap: how do the two groups compare on `median_valid_frac` and `share_zero`? This is an observational comparison, not an experiment. Give one mechanism by which MANRS participation *causes* better ROA coverage, and one by which the causality runs the other way (self-selection). What data would help you tell the two apart?

---

## Task 7: The BGPsec and ASPA Footprint

Detailed guidance: [Task 7 — BGPsec Footprint](Task-7-bgpsec-footprint.md)

### Task 7.1 — Count the path-security artifacts

Run the provided cell, which reports the ROA, VRP, ASPA, and BGPsec router-certificate counts from the live rpki-client snapshot loaded in Task 0.

- [ ] **Q12** Put the two counts side by side: ROAs versus BGPsec router certificates. Explain the gap in terms of what a network *gets* for publishing each one today — one is a unilateral act with immediate benefit, the other buys nothing until neighbors join. Compare the ASPA count as well: ASPA is years younger than BGPsec. What does its count suggest about which kinds of path-security design can actually deploy?

### Task 7.2 — Reflect

Step back from the numbers and consider what the datasets could and could not show.

- [ ] **Q13** Your MANRS list is from today; the routing table and VRPs are from `SNAPSHOT_DATE`. Suppose a network published its first ROAs in the gap between the two dates, or joined MANRS last week. Which direction would each case push your MANRS comparison — overstate or understate the participants' lead?
- [ ] **Q14** This notebook measured *publishing* — MANRS Action 4. The other half of origin validation, networks actually *enforcing* ROV on the routes they hear (Action 1), is invisible in every dataset you used. Why? Sketch how you would measure ROV enforcement from the outside: what would you need to announce, and what would you watch for?

[README](README.md) | [Introduction](Introduction.md) | [Datasets](Datasets.md) | Tasks ⮕ | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)
