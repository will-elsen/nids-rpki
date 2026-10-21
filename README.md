README ⮕ | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)

### [Network Infrastructure Data Science (NIDS) Assignment]

---

# Validating Who May Originate a Prefix: RPKI in Practice

**GitHub:** https://github.com/CAIDA/nids-rpki

## Authors

Will Elsen

## Learning Objectives

This assignment introduces RPKI (Resource Public Key Infrastructure) and Route Origin Validation (ROV), the mechanism that lets routers reject announcements whose origin AS is not authorized to announce a prefix, and the public datasets that show how widely it is deployed: RIPE NCC's daily archive of validated ROA payloads, CAIDA's Routeviews prefix-to-AS table, the MANRS participant list, and rpki-client's live validated RPKI. You will implement ROV yourself, first as a plain scan and then on a prefix index fast enough for the whole routing table, and use it to classify every announced IPv4 route on one day. Afterwards you will be able to explain how ROAs and Max Length defeat prefix and sub-prefix hijacks and where origin validation stops, quantify how much of the internet RPKI vouches for, both per announcement and per network, test whether MANRS participants keep their commitment to publish ROAs, and argue from measured counts why BGPsec has barely deployed while ASPA is spreading.

## Overview

Start by reading **Introduction** to get the background needed to understand the assignment. **Datasets** explains each dataset and how the notebook downloads it.

- step 1 [read the introduction](Introduction.md)
- step 2 [read dataset overviews](Datasets.md)
- step 3 [review the tasks](Tasks.md)
- step 4 log into NRP's JupyterHub, upload and complete the nids-rpki.ipynb
  - Detailed instructions to access NRP: [How to access NRP](https://www.caida.org/projects/nids/how-to/nrp-access/)
  - complete each task by replacing the `# YOUR CODE HERE` sections
  - answer all questions
- step 5 download your working notebook and replace ⬅ deliverable
- step 6 commit and push to github

### Directory Structure

```text
nids-rpki
├- Introduction.md                 # Introduction and background: RPKI, ROV, MANRS, path-security artifacts
├- Datasets.md                     # Dataset overview and access instructions
├- Tasks.md                        # Task checklist and instructions
├- Task-1-concepts.md              # Guidance for Task 1
├- Task-2-rov.md                   # Guidance for Task 2
├- Task-3-internet-scale.md        # Guidance for Task 3
├- Task-4-rov-shares.md            # Guidance for Task 4
├- Task-5-per-as-coverage.md       # Guidance for Task 5
├- Task-6-manrs.md                 # Guidance for Task 6
├- Task-7-bgpsec-footprint.md      # Guidance for Task 7
├- data/                           # The notebook caches its downloaded datasets here
├- nids-rpki.ipynb             ⬅  # Complete / Commit / Push
```

### Glossary
- **AS (Autonomous System)**: A network under a single administrative control (an ISP, cloud provider, university), identified by a globally unique **ASN (AS number)**.
- **AS_PATH**: The ordered list of ASes a route has traversed. Each AS prepends its own ASN as it re-advertises the route; the rightmost ASN is the origin.
- **AS_PATH manipulation (path forgery)**: Advertising a partly or wholly fabricated AS_PATH, for example one that ends in the real owner's ASN so that ROV rates it Valid.
- **ASPA (Autonomous System Provider Authorization)**: An RPKI object in which an AS lists its authorized upstream providers, letting validators detect route leaks and some path forgeries; an AS can publish one unilaterally.
- **BGP (Border Gateway Protocol)**: The path-vector routing protocol that exchanges reachability information between independently operated networks [[RFC 4271](https://www.rfc-editor.org/rfc/rfc4271.html)].
- **BGPsec**: The BGP extension in which every AS on a path signs the route as it forwards it, protecting the entire AS_PATH [[RFC 8205](https://www.rfc-editor.org/rfc/rfc8205.html)].
- **CDF (Cumulative Distribution Function)**: For each value x, the fraction of a sample at or below x; an empirical CDF plots a sample's sorted values against evenly spaced quantiles.
- **CDN (Content Delivery Network)**: A network of servers that caches and serves content close to users; one of the MANRS programs.
- **CSV (Comma-Separated Values)**: A plain-text table format with one record per line; the VRP archive publishes one CSV per trust anchor per day.
- **DDoS (Distributed Denial of Service)**: An attack that floods a target with traffic from many sources, often using forged source addresses.
- **Invalid**: The ROV state of a route that a VRP covers but none authorizes, because of a wrong origin, an excessive prefix length, or both; standard policy drops it.
- **IRR (Internet Routing Registry)**: Databases where operators publish routing policy and route objects, used to build route filters.
- **ISP (Internet Service Provider)**: A network that sells internet connectivity to customers; most network operators in MANRS are ISPs.
- **IXP (Internet Exchange Point)**: A shared facility where many networks interconnect and exchange traffic; one of the MANRS programs.
- **JSON (JavaScript Object Notation)**: A text format for structured data; the MANRS endpoint and rpki-client publish their data as JSON.
- **Longest-prefix match**: The forwarding rule that routers always use the most specific (longest) matching prefix, regardless of AS_PATH length or policy.
- **MANRS (Mutually Agreed Norms for Routing Security)**: A global initiative committing network operators to four baseline practices — filtering, anti-spoofing, coordination, and global validation — with public, measurable accountability [[webpage](https://www.manrs.org/)].
- **MANRS Observatory**: MANRS' public measurement platform, which scores participants against the actions they committed to and publishes the list of participant ASNs [[webpage](https://observatory.manrs.org/)].
- **Max Length**: The ROA field giving the most specific prefix length the origin AS may announce within the ROA's prefix.
- **NIDS (Network Infrastructure Data Science)**: CAIDA's series of assignments on measuring and analyzing internet infrastructure, of which this is one [[webpage](https://www.caida.org/projects/nids/)].
- **NIST (National Institute of Standards and Technology)**: The US standards agency whose RPKI Monitor tracks RPKI adoption over time [[webpage](https://rpki-monitor.antd.nist.gov/)].
- **NOC (Network Operations Center)**: The team or facility that runs a network day to day and is its point of contact for other operators.
- **Not Found**: The ROV state of a route that no VRP covers; standard policy accepts it but ranks it below Valid.
- **NRP (National Research Platform)**: The shared research computing platform whose hosted JupyterHub runs this assignment's notebook [[webpage](https://www.caida.org/projects/nids/how-to/nrp-access/)].
- **PeeringDB**: A public database where networks publish interconnection and contact details.
- **Prefix**: A block of IP addresses (e.g. `203.0.113.0/24`) announced into BGP by the AS that can reach it.
- **Prefix hijack**: An AS announcing a prefix it does not own, so that networks preferring the bogus route send the prefix's traffic to the wrong origin.
- **Prefix-to-AS (pfx2as) dataset**: CAIDA's daily Routeviews-derived table listing every announced IPv4 prefix with the origin AS(es) observed announcing it [[webpage](https://www.caida.org/catalog/datasets/routeviews-prefix2as/)].
- **Radix trie**: A tree keyed on the bits of an address, so that every covering prefix of an address lies on one root-to-leaf path; the structure real validators and routers use for prefix lookups.
- **Relying party**: Software (an RPKI validator) that fetches signed RPKI objects, verifies their signatures and certificate chains, and outputs VRPs for routers.
- **RFC (Request for Comments)**: The numbered document series in which internet standards, including RPKI, ROV, and BGPsec, are published [[webpage](https://www.rfc-editor.org/)].
- **RIR (Regional Internet Registry)**: One of the five registries — ARIN, RIPE NCC, APNIC, LACNIC, AFRINIC — that allocate address space and operate the RPKI trust anchors.
- **ROA (Route Origin Authorization)**: The signed RPKI object declaring an origin ASN, a prefix, and a Max Length.
- **Route leak**: A route propagated in violation of business policy, for example a customer re-announcing one provider's routes to another.
- **Router certificate**: An RPKI certificate binding a router's public key to an ASN, so other routers can verify that AS's BGPsec signatures; the first deployable artifact of BGPsec [[RFC 8209](https://www.rfc-editor.org/rfc/rfc8209.html)].
- **RouteViews**: A University of Oregon project that runs BGP collectors peering with networks worldwide and archives the routes they hear; the source behind CAIDA's pfx2as dataset [[webpage](https://www.routeviews.org/routeviews/)].
- **ROV (Route Origin Validation)**: The check a router runs on every route's (prefix, origin ASN) pair against VRPs, labeling it Valid, Invalid, or Not Found [[RFC 6811](https://www.rfc-editor.org/rfc/rfc6811.html)].
- **RPKI (Resource Public Key Infrastructure)**: The certificate system, rooted in the RIRs, that publishes verifiable, signed records of who may originate which prefixes [[RFC 6480](https://www.rfc-editor.org/rfc/rfc6480.html)].
- **rpki-client**: An open-source RPKI validator (relying party) whose public instance publishes the complete validated global RPKI [[webpage](https://console.rpki-client.org/)].
- **SKI (Subject Key Identifier)**: A hash of a public key that identifies it; each BGPsec router key in the RPKI is listed with its SKI.
- **Sub-prefix hijack**: Announcing a more-specific piece of a victim's prefix so that longest-prefix match wins everywhere, bypassing best-path policy entirely.
- **Trust anchor**: One of the five RIR-operated roots of the RPKI; every VRP validates through exactly one of them.
- **Valid**: The ROV state of a route that at least one VRP authorizes, with a matching origin and a prefix length within Max Length.
- **VRP (Validated ROA Payload)**: A ROA with the cryptography already verified and stripped away — a plain `(prefix, max_length, origin ASN)` tuple that ROV compares routes against.
- **WHOIS**: The registry lookup service that returns the holder and contact details for an IP block or ASN.

README ⮕ | [Introduction](Introduction.md) | [Datasets](Datasets.md) | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)
