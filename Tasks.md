[README](README.md) | Tasks ⮕ | [RPKI](rpki_understanding.md) | [Task 2](Task-rov.md) | [ROV Exercise](rpki_rov_exercise.ipynb) | [MANRS](manrs.md) | [Tasks 4–7](Task-deployment-exercise.md) | [Deployment Exercise](rpki_deployment_exercise.ipynb)

# Tasks

Complete the tasks below in order.

The coding tasks have guidance pages — [Task 2](Task-rov.md) for the ROV exercise and
[Tasks 4–7](Task-deployment-exercise.md) for the deployment exercise — with the
algorithms, worked examples, and common pitfalls. Read the relevant guidance before
writing code.

### Setting up for the notebook tasks

Both notebooks run on an ordinary local Python 3 installation.
In each notebook, start by running the dependency installation cell that begins with
%pip install. This will install the required packages for the notebook.


## Task 1: Read Understanding RPKI

Detailed Guidance → [Task 1](rpki_understanding.md)

Completed in [rpki_understanding.md](rpki_understanding.md). Learn how ROAs and ROV
close the origin-authorization gap.

- [ ] Answer the four *Check your understanding* questions

## Task 2: Implement Route Origin Validation

Detailed Guidance → [Task 2](Task-rov.md)

Completed in [rpki_rov_exercise.ipynb](rpki_rov_exercise.ipynb).

Fill in `classify_rov(announced_prefix, origin_asn, vrps)` so it classifies one BGP
announcement against a DataFrame of VRPs and returns `"Valid"`, `"Invalid"`, or
`"Not Found"`. A plain scan over the rows is fine here — correctness of the three states
matters far more than speed.

- [ ] All 9 hand-built test cases pass
- [ ] The 11 real announcements in `TARGETS` are classified against the 2023-03-01
      snapshot (~400,000 VRPs)
- [ ] Reflection question 1: three targets announce the same address space from the same
      origin at `/18`, `/24`, and `/25` — explain the state boundary in terms of the
      covering ROA's max length
- [ ] Reflection question 2: what does a router with standard policy do with the two
      **Not Found** targets, and why is accepting them still the safe choice today?
- [ ] Reflection question 3: for one **Invalid** announcement, print its covering VRPs
      and identify exactly which condition failed
- [ ] Reflection question 4: why is ignoring the `trust_anchor` column correct?

## Task 3: Read What is MANRS

Detailed Guidance → [Task 3](manrs.md)

Completed in [manrs.md](manrs.md). See how MANRS turns the security mechanisms into
publicly accountable operational practice. No written deliverable — but the deployment
exercise's MANRS comparison (Task 7) builds directly on this reading.

## Task 4: ROV at Internet Scale

Detailed Guidance → [Task 4](Task-deployment-exercise.md#task-4-rov-at-internet-scale)

Completed in [rpki_deployment_exercise.ipynb](rpki_deployment_exercise.ipynb).

The deployment notebook classifies ~1.1 million announced (prefix, origin) pairs against
~1 million VRPs, so a row scan no longer works. The notebook provides a prefix-length
index (`build_vrp_index` / `covering_vrps`); fill in `classify_rov(prefix, origin_asn,
vrp_index)` on top of it. The three-state rule is identical to Task 2 — only the lookup
machinery changes.

- [ ] All 7 hand-built test cases pass
- [ ] `annotate_rov` runs over the full table and produces `bgp_classified` (a minute or
      two on the first run, cached afterwards)

## Task 5: How Much of the Internet Implements RPKI?

Detailed Guidance → [Task 5](Task-deployment-exercise.md#task-5-internet-wide-rov-shares)

Completed in [rpki_deployment_exercise.ipynb](rpki_deployment_exercise.ipynb).

From `bgp_classified`, produce a table with one row per ROV state showing the count and
percentage share of announcements in that state, and a bar chart of the percentage
shares.

- [ ] Q1: Report your headline numbers: what share of announced (prefix, origin) pairs is
      Valid, Not Found, and Invalid? (a) Who is actually affected by the Invalid share,
      and why is it so much smaller than the other two? (b) What does the Not Found share
      say about how much of the internet would go dark if operators dropped Not Found
      routes today?

## Task 6: ROA Coverage, One Network at a Time

Detailed Guidance → [Task 6](Task-deployment-exercise.md#task-6-per-as-roa-coverage)

Completed in [rpki_deployment_exercise.ipynb](rpki_deployment_exercise.ipynb).

Build a DataFrame `per_as`, indexed by origin ASN, with columns `n_announcements` (how
many pairs the AS originates) and `valid_frac` (the fraction of them that are Valid).
Then plot a histogram of `valid_frac` across all origin ASes. Task 7 reuses `per_as`,
so keep the index and column names exactly as specified.

- [ ] Q2: Describe the shape of the histogram. Why does ROA publishing, as an operational
      decision, produce an all-or-nothing distribution? What kind of AS would you expect
      to find in the middle?

## Task 7: MANRS Participants Versus Everyone Else

Detailed Guidance → [Task 7](Task-deployment-exercise.md#task-7-the-manrs-comparison)

Completed in [rpki_deployment_exercise.ipynb](rpki_deployment_exercise.ipynb).

Split the data by whether the origin ASN belongs to an approved MANRS participant, then
produce three things: an announcement-level crosstab of ROV-state percentages for the two
groups; a per-AS summary table (`n_ases`, `median_valid_frac`, `share_zero`,
`share_full`); and a plot comparing the two `valid_frac` distributions (two CDF curves
work well).

- [ ] Q3: Quantify the gap between the two groups on `median_valid_frac` and
      `share_zero`. Give one mechanism by which MANRS participation *causes* better ROA
      coverage, and one by which the causality runs the other way (self-selection). What
      data would help you tell the two apart?

## Task 8: And BGPsec?

Detailed Guidance → [Task 8](rpki_deployment_exercise.ipynb)

Completed in [rpki_deployment_exercise.ipynb](rpki_deployment_exercise.ipynb). The code
for this section is provided: `load_bgpsec_snapshot()` fetches the live validated-RPKI
summary from the rpki-client console and counts ROAs, ASPA records, and BGPsec router
certificates. Your job is to interpret the counts.

- [ ] Q4: Put the ROA count and the BGPsec router-certificate count side by side and
      explain the gap in terms of what a network *gets* for publishing each one. What
      does the ASPA count suggest about which kinds of path-security design can actually
      deploy?
- [ ] Reflection question 1: which direction does the timing mismatch (live MANRS list
      versus archived routing table and VRPs) push your MANRS comparison?
- [ ] Reflection question 2: why is ROV *enforcement* invisible in every dataset you
      used, and how would you measure it from the outside?

[README](README.md) | Tasks ⮕ | [RPKI](rpki_understanding.md) | [Task 2](Task-rov.md) | [ROV Exercise](rpki_rov_exercise.ipynb) | [MANRS](manrs.md) | [Tasks 4–7](Task-deployment-exercise.md) | [Deployment Exercise](rpki_deployment_exercise.ipynb)
