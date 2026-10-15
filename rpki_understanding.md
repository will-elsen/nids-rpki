[README](README.md) | [Tasks](Tasks.md) | RPKI ⮕ | [Task 2](Task-rov.md) | [ROV Exercise](rpki_rov_exercise.ipynb) | [MANRS](manrs.md) | [Tasks 4–7](Task-deployment-exercise.md) | [Deployment Exercise](rpki_deployment_exercise.ipynb)

# Understanding RPKI — Proving Who Owns a Prefix

There are two questions BGP itself can never answer:

1. **Is this AS authorized to originate this prefix?**
2. **Is this AS_PATH a truthful account of how the route actually propagated?**

**RPKI (Resource Public Key Infrastructure)** is the internet's answer to the *first*
question. It doesn't change how BGP announces or selects routes — it adds a separate,
cryptographically signed record of **who is allowed to originate which prefixes**, so that
routers can check an announcement's origin against the truth instead of taking it on faith.

> **Key idea:** RPKI attaches *proof of ownership* to IP address space. The organization
> that legitimately holds a prefix can publish a signed statement — "AS64500 is authorized
> to originate `203.0.113.0/24`" — that anyone on the internet can verify. A hijacker can
> announce whatever they want, but they **cannot forge that signature**, so their false
> origin no longer looks the same as a legitimate one.

RPKI is built on the same trust hierarchy that already governs IP address allocation. The
five **Regional Internet Registries (RIRs** — ARIN, RIPE NCC, APNIC, LACNIC, AFRINIC) hand
out address space, and in RPKI each issues **certificates** binding that address space to
its rightful holder's cryptographic key. Because the authority to sign follows the same
chain as the authority to *allocate*, a signature can be traced back to a trusted root.

---

### Route Origin Authorization (ROA)

A **Route Origin Authorization (ROA)** is the signed statement at the heart of RPKI. A
prefix holder creates a ROA to declare, verifiably, *which AS is allowed to originate their
prefix* — and how specific that AS is allowed to get.

A ROA contains three pieces of information:

```
  ┌──────────────────────────────────────────────────────────┐
  │  ROA                                                       │
  │    Origin ASN:   AS64500          ← who may announce it    │
  │    Prefix:       203.0.113.0/24   ← the address block      │
  │    Max Length:   /24              ← most specific allowed  │
  │                                                            │
  │    (cryptographically signed under the RPKI cert chain     │
  │     that proves the signer holds 203.0.113.0/24)           │
  └──────────────────────────────────────────────────────────┘
```

- **Origin ASN** — the *only* AS authorized to originate this prefix in BGP.
- **Prefix** — the address block being authorized (e.g. `203.0.113.0/24`).
- **Max Length** — the longest (most specific) prefix length the origin may announce. This
  field is subtle but critical, and it's what defends against *sub-prefix* hijacks (below).

The **Max Length** deserves a closer look. If the ROA above sets Max Length to `/24`, then
AS64500 may announce `203.0.113.0/24` — but announcing any *more-specific* piece, like
`203.0.113.128/25`, would **not** be authorized by this ROA. If instead the holder genuinely
wanted to announce `/24` down to `/26` subnets, they'd set Max Length to `/26`. The rule of
thumb operators follow is to set Max Length no larger than they actually announce, so that
no *extra* more-specific announcements are ever considered legitimate.

---

### Route Origin Validation (ROV)

A ROA is just a published record; **Route Origin Validation (ROV)** is the act of *using* it.
ROV is the process a router (helped by a local RPKI validator) runs on every BGP route it
hears: it takes the announcement's **(prefix, origin ASN)** pair and checks it against the
set of published ROAs. The result is one of three states:

```
  For a BGP route with prefix P and origin AS A, compared against the ROAs:

  ┌───────────┬─────────────────────────────────────────────────────────────┐
  │ VALID     │ A ROA covers P, its origin ASN = A, and P's length is within │
  │           │ the ROA's Max Length.        → the origin is authorized      │
  ├───────────┼─────────────────────────────────────────────────────────────┤
  │ INVALID   │ A ROA covers P, but the origin ASN does NOT match A, and/or  │
  │           │ P is more specific than the Max Length allows.               │
  │           │                              → the origin is NOT authorized  │
  ├───────────┼─────────────────────────────────────────────────────────────┤
  │ NOT FOUND │ No ROA covers this prefix at all.                            │
  │ (unknown) │                              → nothing to check against      │
  └───────────┴─────────────────────────────────────────────────────────────┘
```

ROV only *labels* a route — the network operator decides what to do with each label via
**routing policy**. The now-standard practice is:

- **Valid** → accept (prefer it; the origin is proven authorized).
- **Not Found** → accept, but rank it below Valid. (Much of the internet has no ROA yet, so
  outright dropping these would break reachability.)
- **Invalid** → **reject / drop the route.** This is the step that neutralizes hijacks: an
  Invalid route never gets installed and never gets re-advertised onward.

> **Key idea:** ROV doesn't need to know *who* the attacker is or *why* the route is wrong.
> It simply checks the origin against a signed record. A route that contradicts the record
> is dropped — turning a convincing lie into an announcement that responsible routers
> silently refuse to believe.

---

### How RPKI stops Prefix Hijacking

Recall the prefix hijack: the rightful owner **AS64500** announces
`203.0.113.0/24`, and a hijacker **AS66000** announces the *same* prefix with itself as the
origin. Without RPKI, both look equally valid and best-path logic decides who wins.

Now suppose AS64500 has published a ROA — `(203.0.113.0/24, origin AS64500, max /24)` — and
neighboring networks perform ROV:

```
  Announcement heard              ROV check against ROA (origin must = AS64500)     Result
  ────────────────────────────    ───────────────────────────────────────────    ─────────
  203.0.113.0/24  origin AS64500   origin matches, length within max              VALID   → accept
  203.0.113.0/24  origin AS66000   ROA exists but origin AS66000 ≠ AS64500        INVALID → DROP
```

Every ROV-enforcing router that hears the hijacked route marks it **Invalid and drops it**.
The false announcement can't propagate past those routers, so it never wins the best-path
competition on their networks. The hijacker's core trick — *making a false origin look
identical to a real one* — no longer works, because the real one now carries a signature the
hijacker can't reproduce.

---

### How RPKI stops Sub-Prefix Hijacking

The sub-prefix hijack was nastier: the attacker announces a **more-specific** prefix
(`203.0.113.128/25`) so that **longest-prefix-match** wins *everywhere*, bypassing best-path
policy entirely. This is exactly what the ROA's **Max Length** field defends against.

With the ROA `(203.0.113.0/24, origin AS64500, max /24)` in place, ROV evaluates the
more-specific announcement like this:

```
  Announcement heard                 ROV check against ROA (prefix ≤ /24, origin AS64500)   Result
  ───────────────────────────────    ──────────────────────────────────────────────────   ─────────
  203.0.113.128/25  origin AS66000    covered by the ROA, but /25 is MORE specific than     INVALID → DROP
                                      the /24 Max Length  → not authorized
  203.0.113.128/25  origin AS64500    still /25, still beyond the /24 Max Length            INVALID → DROP
```

Notice the second row: **even if the attacker forges the origin to the real owner's ASN**,
the announcement is *still* Invalid, because the ROA never authorized anything more specific
than `/24`. The Max Length closes the more-specific loophole regardless of what origin the
attacker claims. Once the /25 is marked Invalid and dropped, longest-prefix-match has nothing
malicious to prefer.

---

### What RPKI does *not* do

RPKI/ROV answers only the **first** of our two questions — *is this AS authorized to
originate this prefix?* It validates the **origin** of a route, and nothing else. It does
**not** verify the rest of the **AS_PATH**.

That leaves the second question wide open. Consider the **AS_PATH manipulation** attack: a
hijacker forges the path so that it *ends in the real owner's ASN* (AS64500). To ROV, the
origin looks correct — the rightmost ASN is the authorized one — so the route can pass as
**Valid** even though the path is a lie and traffic is being diverted through the attacker.

```
  Forged route:  203.0.113.0/24   AS_PATH = [66000 ... 64500]
                                              ▲            ▲
                                              │            └─ origin AS64500 → ROV says VALID ✓
                                              └─ attacker inserted itself into the path
                                                 (ROV never checks this part)
```

This is the gap **BGPsec** is designed to close: cryptographically protecting the *entire
AS_PATH*, not just the origin. RPKI and BGPsec are complementary — RPKI proves *who may
originate a prefix*, and BGPsec proves *that the path really propagated the way it claims*.

---

### From signed ROAs to Validated ROA Payloads (VRPs)

Everything above describes ROAs as signed cryptographic objects. In practice a router never
handles those signatures itself. A **relying party** (an RPKI validator) does that work: it
fetches all the signed objects from the repositories, verifies every signature and
certificate chain, and emits the result as a flat list of plain, already-validated tuples
called **Validated ROA Payloads (VRPs)**:

```
  (prefix, max_length, origin ASN)
```

A VRP is simply a ROA with the cryptography already checked and stripped away. This flat
list is what ROV actually compares routes against — the three-state check from earlier is a
lookup over a table of VRPs.

**Trust anchors.** RPKI is rooted in five **trust anchors**, one operated by each RIR. Every
VRP traces back through a certificate chain to exactly one of them:

| trust anchor | RIR |
| --- | --- |
| `arin` | ARIN |
| `ripencc` | RIPE NCC |
| `apnic` | APNIC |
| `lacnic` | LACNIC |
| `afrinic` | AFRINIC |

Crucially, the trust anchor a VRP came from tells you *which RIR's chain validated it* — it
is useful for grouping or attributing VRPs by RIR, but it plays **no part in the validation
decision**. A route's state depends only on **prefix coverage**, **origin AS**, and **max
length**.

Because a VRP is just data, the whole validated state of RPKI at any moment is nothing more
than a large table of these tuples — and ROV becomes a lookup against that table.

---

### Check your understanding

1. In one sentence each, what is a **ROA** and what is **ROV**?
2. A router runs ROV on a route and gets **Not Found**. Why is dropping every Not Found route
   a bad idea today?
3. An operator announces `203.0.113.0/24` and also, sometimes, `203.0.113.0/26`. What should
   the **Max Length** on their ROA be, and what would break if they set it to `/24`?
4. Why can a **prefix hijack** be stopped by RPKI, while a **forged-AS_PATH** attack that
   ends in the real origin can slip through as Valid?

---

### RPKI resources
- [Cloudflare — RPKI: the required cryptographic upgrade to BGP routing](https://blog.cloudflare.com/rpki/) — approachable intro to why BGP needed origin validation.
- [RPKI Read the Docs — Securing BGP](https://rpki.readthedocs.io/en/latest/rpki/securing-bgp.html) — thorough free reference on RPKI, ROAs, and ROV.
- [RFC 6480 — An Infrastructure to Support Secure Internet Routing](https://www.rfc-editor.org/rfc/rfc6480.html) — the RPKI specification.
- [RFC 6811 — BGP Prefix Origin Validation](https://www.rfc-editor.org/rfc/rfc6811.html) — defines how routers perform ROV.

[README](README.md) | [Tasks](Tasks.md) | RPKI ⮕ | [Task 2](Task-rov.md) | [ROV Exercise](rpki_rov_exercise.ipynb) | [MANRS](manrs.md) | [Tasks 4–7](Task-deployment-exercise.md) | [Deployment Exercise](rpki_deployment_exercise.ipynb)
