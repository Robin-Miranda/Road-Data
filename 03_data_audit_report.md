---
title: "D2 Data audit report and data-quality report (draft v0.1)"
project: "Optimal Placement of EV Charging Stations for Intercity Government Buses in Karnataka Using Geospatial Analytics and Mathematical Optimization"
deliverable: "ANCP D2, due Fri 23 Oct 2026, 17:00 IST (PDF ≤ 8 pages plus a 3–4 page datasheet appendix)"
dataset_version: "data-v1.0"
date: "2026-10-20"
note: "Draft content to be laid out in the course template: cover page, A4, 11 pt, numbered and captioned figures and tables, APA 7 references."
---

> **Update (v1.2, 2026-10-22):** v1.2 adds the team-downloaded KSRTC division route km (`ksrtc_divisions_route_km_2021.csv`) and the MoP state counts (`mop_public_evcs_statewise_2025-08.csv`). It also upgrades the tariff parameters to verified values from the KERC order, including a KSRTC-depot time-of-day tariff that makes overnight charging ₹3.50/kWh from December to June. Submit **v1.2** as the D2 dataset.
>
> **Update (v1.1, 2026-10-20):** v1.1 adds a name-based `bus_facility_class` to the bus-station candidates. Of the 2,115 Overture "bus stations", only 404 are terminals or depots; 1,589 are bus stops and 17 are misclassified ATMs, pharmacies and hotels. v1.1 also adds a graph-node table. Every count in this report is unchanged. (Superseded by v1.2.)

# 1. Purpose and scope of the data

The model has to decide where to place charging sites for KSRTC intercity electric buses, and how many chargers to put at each site. To make that decision it needs five kinds of input: (a) the routes buses drive and their distances, (b) the candidate sites (depots and terminals, plus highway sites), (c) the chargers that already exist, (d) the energy behaviour of the buses, and (e) costs. Phase One searched for data on all five. It downloaded and profiled everything that is openly obtainable, and it registered the rest with an honest verification level (see `01_source_register.md`).

**Default pilot corridors.** The six Bengaluru routes on which KSRTC already runs AC e-buses: Mysuru, Chikkamagaluru, Shivamogga, Madikeri, Virajpet and Davanagere (source G4). They are real, operating, and short enough to model in full. The data pipeline covers the whole state, so the method extends to a statewide network without new data work.

# 2. Source and access

| Dataset | Origin | Licence / permission | Accessed |
|---|---|---|---|
| Overture Maps 2026-09-23.0: roads, places, divisions | Public AWS S3 bucket, read anonymously | ODbL 1.0 (roads, divisions); CDLA-Permissive-2.0 (places). Attribution: "© OpenStreetMap contributors, Overture Maps Foundation". | 2026-10-20 |
| Open Charge Map India export | github.com/openchargemap/ocm-export, commit 8e3bedca (2026-04-22) | CC BY 4.0 for all 379 Karnataka records | 2026-10-20 |
| Parameters register (bus, tariff, cost) | Hand-entered from cited public documents and press | Citation only; no data is redistributed | 2026-10-20 |

Three official sources were downloaded by the team on 2026-10-22, because the analysis environment cannot reach those sites. They are stored unchanged in `data/raw/manual/`:

| Dataset | Origin | Licence / permission | Accessed |
|---|---|---|---|
| KSRTC Divisionwise Route and route kms as on 31-03-2021 (CSV) | data.gov.in | GODL | 2026-10-22 |
| MoP state-wise public EVCS annexure (PDF; table extracted by `src/extract_mop_evcs.py`) | static.pib.gov.in | Public record | 2026-10-22 |
| KERC Combined Tariff Order 2025 (text of the PDF) | KERC | Public regulatory order | 2026-10-22 |

**Checks on these files.** The KSRTC division rows sum exactly to the corporation total of 5,471 routes and 426,382 km, and each average equals km ÷ routes (maximum error 0.005 km). The MoP state counts sum to the printed national total of 29,277, and Karnataka is **6,097**. The KERC order confirms the EV tariff of **₹4.50/kWh** (Annexure-9, p.542). Other official sources (BEE guidelines, Kaggle, Geofabrik) remain at level B or C in the register.

# 3. Consent and purpose limitation (DPDP Act, 2023)

None of the datasets holds personal data about passengers, employees or private individuals. The OCM export contains public business locations. The cleaning script keeps no user comments, contributor names or contact fields from OCM. The Overture places used here are businesses and transport facilities; no person-level records are kept. Because the project processes no digital personal data, the DPDP Act's consent and notice obligations do not apply. If KSRTC later supplies ticketing or crew data, only aggregated route-level counts will be requested, and their purpose will be limited to infrastructure planning.

# 4. Quality audit

Every figure in this section comes from `data/processed/v1.0/quality_profile.json`.

## 4.1 Missing values

| Dataset | Field | Missing | Treatment and reason |
|---|---|---|---|
| Roads (116,389 segments) | name | 60.2% | Kept as null. Names are labels only; the model relies on topology and length. |
| | route_refs (NH/SH number) | 50.9% | Kept as null. Used only for map labels and corridor selection, where gaps are filled by the graph path. |
| | max_speed_kmh | 89.5% | **Not used** for travel time. In D3, impute speed by road class from a stated assumption, and test it in sensitivity. |
| | surface | 61.9% | Kept as null. Major roads are assumed paved. |
| Towns (24,807) | population | 98.7% (only 329 populated) | Do not use population as demand. Demand comes from KSRTC trips (request G1/G6). Population is a tie-break only, for cities and towns. |
| | settlement_class | 372 | Kept as null; excluded from the origin–destination town list. |
| OCM stations (379) | town / state / postcode | 2 / 3 / 1 | Kept. District comes from coordinates, not from the text fields. |
| | max_power_kw | 1 | Kept as null; the site is not flagged as bus-capable. |
| Candidate sites (5,458) | operating_status | 100% | Field unused in this release. Confidence is used instead. |
| Parameters (25) | value | 5 gaps (usable SoC, reserve, charger unit cost, O&M, site rental) | **Explicit modelling assumptions** with ranges, flagged in the experiment log and varied in sensitivity. |

## 4.2 Duplicates

| Check | Result | Treatment |
|---|---|---|
| Duplicate IDs (all tables) | 0 | — |
| Towns with the same name and identical coordinates | 1 | Dropped |
| OCM sites within 50 m of another OCM site | 8 | **Flagged, not dropped.** Neighbouring sites of different operators are real (e.g. two networks in one mall), so they are kept and the column `near_duplicate_50m` lets the model merge them into one candidate. |
| Overture bus stations within 50 m of a higher-confidence one | 19 | Flagged. Merge into a single candidate site at the modelling stage. |
| Overture fuel stations within 50 m | 105 | Flagged, same treatment |
| Overture EV stations vs OCM | 51 vs 379 | OCM is the primary source; Overture EV points serve only as a cross-check. |

## 4.3 Outliers and validity

| Check | Result | Treatment |
|---|---|---|
| Points outside Karnataka | All layers were clipped to the state polygon. 14 OCM sites lie inside the state but have a different state text; 1 site says "Karnataka" but lies outside. | The coordinate test is used, not the state text. |
| Charger power > 500 kW | 0. The maximum is 240 kW (1 site); the median is 60 kW. | None needed |
| Road segments shorter than 1 m | 9 | Kept (they are topology stubs at junctions) and have no effect on distances. |
| Road segments longer than 20 km | 26 (up to 219 km) | Inspected: these are the access-controlled Hyderabad–Panaji (EC 10) and Solapur–Chennai (NH 150C) economic corridors, which have few junctions. Kept, but **flagged for D3 review**, because these corridors may be partly under construction. |
| Staleness of OCM records | 340 of 379 last verified in 2025 and 20 in 2026; 11 not since 2018 | Kept. Status is used as given; records not verified since before 2024 are flagged in EDA. |
| Low-confidence places (< 0.5) | 52 bus stations, 258 fuel stations | Kept but marked `low_confidence`. They are excluded from the main candidate set and added back in a sensitivity run. |

## 4.4 Fitness of the road graph

- **Connectivity.** The major-road graph has 161,526 nodes and 173 components. The largest component holds 99.0% of nodes and 97.4% of road km, so routing is feasible across the state.
- **Plausibility test** (`routing_smoke_test.csv`). Shortest-path distances from Bengaluru are: Mysuru 140 km, Davanagere 260 km, Madikeri 255 km, Chikkamagaluru 231 km, Shivamogga 275 km, Mangaluru 345 km, Hubballi 404 km and Belagavi 500 km. Circuity (network distance ÷ straight-line distance) runs from 1.06 to 1.24, the usual range for intercity road networks. The town points snap to the graph within 43–333 m. **Next step:** compare the six pilot routes with KSRTC's published route km once G1 is downloaded, and record the error.
- **Known limits.** The graph is undirected and has no turn restrictions. Roads below secondary class are dropped, so detours into town-centre bus stands may be under-estimated by the length of the final local stretch. D4 will add a snapping penalty for this.

# 5. Representativeness and bias

1. **Existing chargers are heavily under-counted.** OCM lists 379 public sites in Karnataka, while the Ministry of Power reports 6,097 public EVCS (E3, Aug 2025). That is about 6% coverage. 158 of the 379 sites (42%) are in Bengaluru Urban. Conclusions about "existing infrastructure we can reuse" are therefore a lower bound, and they are biased toward Bengaluru and toward networks that publish to OCM (Tata Power, Statiq, Zeon, Jio-bp, ChargeZone).
2. **Existing chargers are built for cars.** Only 45 sites are ≥ 120 kW and only 1 is ≥ 240 kW. CCS2 is common (343 sites), but power and bay geometry rarely suit a 12 m coach. The model therefore treats public chargers as context, not as capacity for buses.
3. **Bus stations are a mix of facility types.** The 2,115 "bus stations" combine KSRTC terminals with city and private stops, and 98% come from one upstream source (Meta). Kodagu has 12 and Chamarajanagar 14, against 65 in Hassan, which likely reflects mapping effort rather than service. Bus-station density should not be read as demand.
4. **The bus specifications come from one manufacturer, in press coverage.** Battery and range figures come from Olectra launch material and third-party sites, and the two sources disagree (360 vs 422 kWh). The kWh/km figure is a tender allowance tested with the AC off. The likely bias is optimistic range, so sensitivity tests must run toward higher consumption.
5. **Costs come from other states.** The capex benchmarks are from Delhi and Punjab tenders, and their scopes differ. They give orders of magnitude, not Karnataka prices.

# 6. Feature engineering log

| Feature | Logic | Leakage risk |
|---|---|---|
| `length_m` (segments, edges) | Haversine sum over the vertices; an edge takes the connector fraction × segment length | None. This is a physical property. |
| `in_giant_component` | The segment touches a node of the largest connected component | None |
| `route_refs`, `surface`, `max_speed_kmh` | Flattened from Overture nested lists | None |
| `district` (all point layers) | Point-in-polygon against the 31 Overture district polygons | None |
| `max_power_kw`, `has_dc`, `has_ccs2`, `bus_capable_dc_ge_60kw` | Aggregated over a site's connectors | None, but the 60 kW threshold is a screening choice to revisit |
| `near_duplicate_50m` | Grid-bucketed haversine test, ordered by confidence | None |
| `low_confidence` | Overture confidence < 0.5 | None |
| `circuity` | Network km ÷ straight-line km | None |

This is an optimisation project, not a predictive one, so "leakage" here means letting the output decide the inputs. One example would be choosing candidate sites with knowledge of where KSRTC later installed chargers. The candidate set is therefore fixed in v1.0, before any modelling, and any later change gets a new version tag.

# 7. What is ready, and what the team must still do before Friday

**Ready (data-v1.2):** a reproducible raw-to-clean pipeline, the cleaned tables, checksums, the data dictionary and the quality profile.

**To do before submission:**
1. ~~Download G1, E3 and the KERC order~~ (done 2026-10-22; now level A).
2. Confirm the Overture licence lines at docs.overturemaps.org/attribution.
3. Lay out sections 1–6 and the datasheet (Appendix A) in the course template, at 8 pages or fewer.

# Appendix A. Datasheet for data-v1.0 (after Gebru et al., 2021)

**Motivation.** Created for an ANCP capstone (XIME, Term 5, 2026) to support a charging-site placement decision for KSRTC intercity e-buses. It was assembled by the student team; there is no external funding.

**Composition.** Seven tables: 379 existing public charging sites with 585 connectors; 5,458 candidate sites (2,115 bus stations, 3,291 fuel stations, 51 EV points, 1 rest stop); 116,389 major-road segments forming a graph of 161,526 nodes and 169,153 edges; 24,807 settlements; 31 districts; the state boundary; and 25 cited parameters. There are no personal data and no sampling: every layer is the full public extract inside the state. Known gaps are the absence of KSRTC route, schedule and depot data, and missing values for usable SoC, reserve, O&M and charger unit cost.

**Collection process.** Overture GeoParquet was read anonymously from S3 with a bounding-box filter (lon 73.9–78.7, lat 11.4–18.6) and then clipped to the state polygon. OCM was taken from a git sparse checkout of `data/IN`. The parameters were entered by hand from documents found through web search on 2026-10-20, each with its URL and a verification level.

**Preprocessing.** Clipping, deduplication of exact duplicates, flags for near-duplicates and low confidence, flattening of nested fields, geodesic lengths, graph construction and district tagging. Raw files are kept unchanged in `data/raw/`.

**Uses.** Candidate generation, route-distance and detour computation, and benchmarking against existing chargers for the D3–D5 optimisation. It is **not suitable** for estimating passenger demand, for travel-time modelling until speeds are imputed, or for publishing final costs without checking the parameters.

**Distribution.** The team repository, tagged `data-v1.0`. Derived data must keep the ODbL share-alike and attribution terms (roads, divisions) and CC BY 4.0 attribution (OCM).

**Maintenance.** The team owns the dataset until the end of the course. A rerun of `src/fetch_overture.py` with a new `RELEASE` constant, or a new OCM commit, produces data-v1.1. The checksums detect any silent change.

# References (APA 7; to be completed)

- Gebru, T., Morgenstern, J., Vecchione, B., Vaughan, J. W., Wallach, H., Daumé III, H., & Crawford, K. (2021). Datasheets for datasets. *Communications of the ACM, 64*(12), 86–92.
- Open Charge Map. (2026). *ocm-export* [Data set, commit 8e3bedca]. GitHub. https://github.com/openchargemap/ocm-export
- Overture Maps Foundation. (2026). *Overture Maps release 2026-09-23.0* [Data set]. https://overturemaps.org
- Government of India. (2023). *The Digital Personal Data Protection Act, 2023*. MeitY.
- Additional sources are listed with URLs in `01_source_register.md` and `parameters_register.csv`.
