README ⮕ | [Tasks](Tasks.md) | [RPKI](rpki_understanding.md) | [Task 2](Task-rov.md) | [ROV Exercise](rpki_rov_exercise.ipynb) | [MANRS](manrs.md) | [Tasks 4–7](Task-deployment-exercise.md) | [Deployment Exercise](rpki_deployment_exercise.ipynb)

### Network Infrastructure Data Science (NIDS) Module

---

# RPKI — Validating Who May Originate a Prefix

## Learning Objectives

The goal of this module is to:
- learn how RPKI closes the origin-authorization gap in BGP — ROAs, Route Origin Validation, and Max Length — and implement ROV yourself against a real snapshot of validated RPKI data
- see how MANRS turns these mechanisms into publicly accountable operational practice
- measure real-world deployment for yourself — run ROV across a full day of the global routing table, quantify how much of the internet RPKI vouches for, test whether MANRS participants keep their ROA-publishing commitment, and count how many networks have published a BGPsec router certificate

## Overview

This module assumes you are comfortable with how BGP works and how prefix hijacks happen. Start by reading **Understanding RPKI**. Each file contains marked sections and questions for you to fill in as you go.

- step 1 [review the tasks index](Tasks.md) — the ordered list of tasks (1–8) across the readings and notebooks, with what is scaffolded versus what you implement
- step 2 [read understanding RPKI](rpki_understanding.md) — how ROAs and ROV neutralize origin hijacks; answer the *Check your understanding* questions (Task 1)
- step 3 complete the [ROV exercise notebook](rpki_rov_exercise.ipynb) — implement Route Origin Validation and classify real announcements against a real RPKI snapshot (Task 2); read the [Task 2 guidance](Task-rov.md) first
- step 4 [read what is MANRS](manrs.md) — the operational commitments that put these mechanisms into everyday practice (Task 3)
- step 5 complete the [deployment exercise notebook](rpki_deployment_exercise.ipynb) — classify every announced route on the internet against a real RPKI snapshot, compare MANRS participants with everyone else, and count BGPsec's real-world footprint (Tasks 4–8); guidance page: [Tasks 4–7](Task-deployment-exercise.md)
- step 6 commit and push your completed module to github

### Directory Structure

```
rpki_module
├- Tasks.md                         # Index of every task in this module (Tasks 1-8)
├- rpki_understanding.md         ⬅  # RPKI, ROAs, and Route Origin Validation (ROV)
├- Task-rov.md                      # Guidance - implementing classify_rov (Task 2)
├- rpki_rov_exercise.ipynb       ⬅  # Notebook - implement ROV against a real RPKI snapshot
├- manrs.md                         # MANRS - the operational norms layer on top of the mechanisms
├- Task-deployment-exercise.md      # Guidance - the deployment-exercise coding tasks (Tasks 4-7)
└- rpki_deployment_exercise.ipynb⬅  # Notebook - measure real-world RPKI/BGPsec deployment, MANRS vs the rest
```

### Glossary

- **BGP (Border Gateway Protocol)**: The path-vector routing protocol that exchanges reachability information between independently operated networks, deciding how traffic moves across the internet.
- **Autonomous System (AS)**: A network under a single administrative control (an ISP, cloud provider, university), identified by a globally unique **AS number (ASN)**.
- **Prefix**: A block of IP addresses (e.g. `203.0.113.0/24`) announced into BGP by the AS that can reach it.
- **Origin AS**: The AS that originates a prefix announcement — the rightmost ASN in the AS_PATH, and the ASN that RPKI checks for authorization.
- **Prefix hijack**: An AS announcing a prefix it does not own, so that networks preferring the bogus route send the prefix's traffic to the wrong origin.
- **Sub-prefix hijack**: Announcing a *more-specific* piece of a victim's prefix so that longest-prefix match wins everywhere, bypassing best-path policy entirely.
- **Longest-prefix match**: The forwarding rule that routers always use the most specific (longest) matching prefix, regardless of AS_PATH length or policy.
- **RPKI (Resource Public Key Infrastructure)**: A cryptographic system layered alongside BGP that publishes verifiable, signed records of who is authorized to originate which prefixes.
- **ROA (Route Origin Authorization)**: The signed RPKI object declaring an origin ASN, a prefix, and a **Max Length** — the most specific announcement the origin is authorized to make.
- **ROV (Route Origin Validation)**: The check a router runs on every route's (prefix, origin ASN) pair against published ROAs, labeling it **Valid**, **Invalid**, or **Not Found** — with Invalid routes dropped.
- **VRP (Validated ROA Payload)**: A ROA with the cryptography already verified and stripped away — a plain `(prefix, max_length, origin ASN)` tuple that ROV actually compares routes against.
- **RIR (Regional Internet Registry)**: The five registries (ARIN, RIPE NCC, APNIC, LACNIC, AFRINIC) that allocate address space and operate the RPKI trust anchors.
- **MANRS (Mutually Agreed Norms for Routing Security)**: A global initiative committing network operators to four baseline practices — filtering, anti-spoofing, coordination, and global validation — with public, measurable accountability.
- **MANRS Observatory**: MANRS' public measurement platform, which scores participants against the actions they committed to — including a public list of participant ASNs.
- **Prefix-to-AS (pfx2as) dataset**: A daily CAIDA/Routeviews-derived table listing every announced prefix with the origin AS(es) observed announcing it — a compact snapshot of the global routing table, and the "what is announced" half of deployment measurement.
- **BGPsec**: The extension that cryptographically protects the *entire AS_PATH*, closing the path-validation gap that RPKI/ROV leaves open.
- **Router certificate**: An RPKI certificate binding a public key to an ASN, so any router can verify that AS's BGPsec signatures.
- **ASPA (Autonomous System Provider Authorization)**: A newer RPKI object in which an AS lists its authorized upstream providers, targeting route leaks — and, unlike BGPsec, deployable unilaterally.

README ⮕ | [Tasks](Tasks.md) | [RPKI](rpki_understanding.md) | [Task 2](Task-rov.md) | [ROV Exercise](rpki_rov_exercise.ipynb) | [MANRS](manrs.md) | [Tasks 4–7](Task-deployment-exercise.md) | [Deployment Exercise](rpki_deployment_exercise.ipynb)
