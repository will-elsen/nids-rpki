[README](README.md) | [Introduction](Introduction.md) | Datasets ⮕ | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)

# Datasets

The notebook downloads all four datasets directly from public archives in its Task 0 section
and caches what it keeps under `data/`. Nothing needs credentials, and later runs read the
cached copies.

| dataset | answers | used in | snapshot |
| --- | --- | --- | --- |
| RIPE NCC validated ROA payload archive | What is **authorized**? | Tasks 2–6 | 2023-03-01 (Task 2) and 2026-07-01 (Tasks 3–6) |
| CAIDA Routeviews prefix-to-AS | What is **announced**? | Tasks 3–6 | 2026-07-01 |
| MANRS participant ASNs | Who has **publicly committed**? | Task 6 | live, cached on first run |
| rpki-client validated output | How far has **path security** got? | Task 7 | live, cached on first run |

The routing table and the VRPs are pinned to the same archived day, `SNAPSHOT_DATE`, so the
comparison is internally consistent. The MANRS list and the rpki-client summary are fetched
live, because no public historical archive of either exists.

#### RIPE NCC Validated ROA Payload (VRP) Archive

RIPE NCC publishes a daily historical archive of validated ROA payloads. Despite being hosted
by RIPE NCC, it contains the validated output of **all five** trust anchors. One
xz-compressed CSV is published per trust anchor per day:

```
https://ftp.ripe.net/ripe/rpki/<ta>.tal/<YYYY>/<MM>/<DD>/roas.csv.xz

<ta> is one of:  afrinic, ripencc, arin, lacnic, apnic
```

> **Gotcha:** the identifier for the RIPE NCC trust anchor is `ripencc`, not `ripe`.

The raw files have these columns:

| Name | Type | Description |
| --- | --- | --- |
| `URI` | string | rsync location of the signed ROA object the VRP came from |
| `ASN` | string | authorized origin AS, written `AS7377` |
| `IP Prefix` | string | authorized prefix, e.g. `137.110.0.0/16` |
| `Max Length` | integer | most specific announcement allowed; blank means the prefix's own length |
| `Not Before` | timestamp | start of the ROA's validity window |
| `Not After` | timestamp | end of the ROA's validity window |

`load_rpki_snapshot(date)` decompresses and normalizes the five files for a day into one
DataFrame with one row per VRP. The 2023-03-01 snapshot (`rov_vrps`) holds about 400,000
VRPs; the 2026-07-01 snapshot (`vrps`) holds close to a million.

| Name | Type | Description |
| --- | --- | --- |
| `prefix` | string | authorized prefix, e.g. `137.110.0.0/16` |
| `max_length` | int | most specific announcement allowed, e.g. `20` |
| `asn` | int | AS authorized to originate the prefix, e.g. `7377` |
| `trust_anchor` | string | trust anchor that validated the VRP: `arin`, `ripencc`, `apnic`, `lacnic`, `afrinic` |
| `family` | int | IP address family, `4` or `6` |
| `date` | string | snapshot date the rows were loaded for, e.g. `2026-07-01` |

#### CAIDA Routeviews Prefix-to-AS (pfx2as)

CAIDA's **Routeviews prefix-to-AS** dataset condenses a day of global BGP routing-table
snapshots from RouteViews collectors into exactly what ROV needs: every announced IPv4
prefix, together with the origin AS observed announcing it
[ [webpage](https://www.caida.org/catalog/datasets/routeviews-prefix2as/) | [download](https://publicdata.caida.org/datasets/routing/routeviews-prefix2as/) ].

```
https://publicdata.caida.org/datasets/routing/routeviews-prefix2as/
        <YYYY>/<MM>/routeviews-rv2-<YYYYMMDD>-1200.pfx2as.gz
```

Each line of the raw file is tab-separated:

| Name | Type | Description |
| --- | --- | --- |
| network | string | network address, e.g. `35.228.224.0` |
| prefix length | int | combine with the network: `35.228.224.0/19` |
| origin AS | string | observed origin AS, e.g. `15169`; several origins are joined with `_` (`64500_64501`) |

`load_prefix2as_snapshot(date)` returns `bgp`, one row per announced **(prefix, origin)
pair** — a prefix observed with two origins becomes two rows, because ROV classifies each
pair separately. On 2026-07-01 that is roughly 1.1 million rows. The `rv2` files cover IPv4
only.

| Name | Type | Description |
| --- | --- | --- |
| `prefix` | string | announced prefix, e.g. `1.0.0.0/24` |
| `origin` | int | observed origin AS, e.g. `13335` |
| `date` | string | snapshot date, e.g. `2026-07-01` |

#### MANRS Participant ASNs

The MANRS Observatory exposes a small public endpoint returning the ASNs claimed by
**approved MANRS participants** across all of its programs
[ [webpage](https://www.manrs.org/) | [observatory](https://observatory.manrs.org/) ]:

```
https://observatory.manrs.org/api/v2/asns
```

| Name | Type | Description |
| --- | --- | --- |
| `asns` | list of int | every ASN claimed by an approved participant |

`load_manrs_asns()` returns the list as a Python `set`, `manrs_asns` (about 1,450 ASNs). The
endpoint reports **current** membership, not membership as of `SNAPSHOT_DATE`.

#### rpki-client Validated RPKI Output

The `rpki-client` project runs a public validator and publishes its complete validated
output — every VRP, ASPA record, and BGPsec router key in the global RPKI — as one JSON
document of about 100 MB, refreshed every few minutes
[ [webpage](https://console.rpki-client.org/) | [download](https://console.rpki-client.org/rpki.json) ].

`load_bgpsec_snapshot()` downloads it once and keeps only these fields, as `snapshot`:

| Name | Type | Description |
| --- | --- | --- |
| `buildtime` | string | when the validator produced the output, e.g. `2026-10-15T15:35:30Z` |
| `roas` | int | number of validated ROA objects |
| `vrps` | int | number of VRPs those ROAs expand to |
| `aspas` | int | number of validated ASPA records |
| `bgpsec_keys` | list of dict | one entry per BGPsec router key: `asn`, `ski` (subject key identifier), `pubkey`, `ta` (trust anchor), `expires` (Unix time) |

[README](README.md) | [Introduction](Introduction.md) | Datasets ⮕ | [Tasks](Tasks.md) | [Task 1](Task-1-concepts.md) | [Task 2](Task-2-rov.md) | [Task 3](Task-3-internet-scale.md) | [Task 4](Task-4-rov-shares.md) | [Task 5](Task-5-per-as-coverage.md) | [Task 6](Task-6-manrs.md) | [Task 7](Task-7-bgpsec-footprint.md) | [Notebook](nids-rpki.ipynb)
