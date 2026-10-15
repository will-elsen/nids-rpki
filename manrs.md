[README](README.md) | [Tasks](Tasks.md) | [RPKI](rpki_understanding.md) | [Task 2](Task-rov.md) | [ROV Exercise](rpki_rov_exercise.ipynb) | MANRS ⮕ | [Tasks 4–7](Task-deployment-exercise.md) | [Deployment Exercise](rpki_deployment_exercise.ipynb)

# What is MANRS?

**MANRS (Mutually Agreed Norms for Routing Security)** is a global initiative that asks
network operators to commit to a small set of concrete routing-security practices. It is
**not a protocol**. There is nothing to install and nothing on the wire — MANRS is an agreed
baseline of *behavior*, plus public accountability for following it.

> **Key idea:** Most routing incidents are not exotic attacks; they are ordinary mistakes that
> propagate because nobody along the way was filtering. MANRS' premise is that if enough
> networks independently commit to a few basic hygiene practices, the whole system gets
> measurably safer — no flag day, no new protocol, no waiting on anyone else.

Launched in 2014 and now run by the **Global Cyber Alliance** (originally the Internet
Society), MANRS organizes participants into **programs** — network operators (ISPs), IXPs,
CDN/cloud providers, and equipment vendors — each with actions suited to that role.

---

### The four core actions

For a network operator, participation means committing to these:

| Action | What it means |
| --- | --- |
| **1. Filtering** | Don't accept or propagate incorrect routing announcements. Define what your customers may announce and filter everything else — the single biggest defense against hijacks and leaks. |
| **2. Anti-spoofing** | Prevent packets with forged source IP addresses from leaving your network, so it can't be used to launch reflection/amplification DDoS attacks. |
| **3. Coordination** | Keep contact information current and globally accessible (PeeringDB, RIR whois, your own NOC page) so other operators can reach you when something goes wrong. |
| **4. Global validation** | Publish your routing data so others can validate it — accurate IRR records and, above all, **published ROAs** so other networks can perform origin validation on your prefixes. |

---

### How it relates to RPKI and BGPsec

RPKI, ROV, and BGPsec are **mechanisms** — the cryptography and protocol machinery. MANRS is
the **social layer** on top of them: it turns "this technology exists" into "this network has
publicly committed to using it, and can be checked."

Action 4 is where the connection is most direct. Publishing ROAs is only half of origin
validation — the other half is other networks actually validating against them, which is
Action 1. MANRS gets both halves adopted at once, by getting many operators to commit
together.

---

### Why it matters

- **It's incrementally deployable.** Every action is unilateral. A network gains value from
  doing it regardless of what its neighbors do.
- **It creates reputational pressure.** Participants are listed publicly, and the **MANRS
  Observatory** scores networks against the actions — so compliance is visible, and
  increasingly appears in peering and procurement conversations.
- **It targets the common case.** Most real-world outages come from fat-fingered
  announcements and missing filters, not from adversaries with forged AS_PATHs. MANRS attacks
  that majority directly.

---

### Resources
- [MANRS — official site](https://www.manrs.org/) — actions, participant list, and how to join.
- [MANRS Observatory](https://observatory.manrs.org/) — public measurement of how networks score against the actions.

[README](README.md) | [Tasks](Tasks.md) | [RPKI](rpki_understanding.md) | [Task 2](Task-rov.md) | [ROV Exercise](rpki_rov_exercise.ipynb) | MANRS ⮕ | [Tasks 4–7](Task-deployment-exercise.md) | [Deployment Exercise](rpki_deployment_exercise.ipynb)
