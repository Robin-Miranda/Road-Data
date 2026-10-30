---
title: "Source register: Phase One data discovery"
project: "Optimal Placement of EV Charging Stations for Intercity Government Buses in Karnataka"
deliverable: "ANCP D2 (input to the data audit report)"
version: "data-v1.0"
date_accessed: "2026-10-20"
---

# Source register (Phase One)

Every source the team found is listed here, including the ones that could not be downloaded. Nothing in this register is invented: each row says how far it was verified.

**Verification levels used below**

| Level | Meaning |
|---|---|
| **A. Downloaded and profiled** | The file is in `data/raw/`, and every figure about it in this project was computed from the file itself. |
| **B. Official document located, not opened** | The publisher and URL come from web search results. The file could not be opened from the analysis environment, because the network blocks `data.gov.in`, `kerc.karnataka.gov.in`, `beeindia.gov.in` and most `.gov.in` hosts. Its figures come from search snippets or press coverage, and the team must check them in the document before citing them. |
| **C. Secondary report only** | The source is a news, vendor or third-party page. Use it only as a scenario value, and cite it as a secondary source. |
| **D. Not found** | No usable public source was found. The value has to be an explicit modelling assumption, or requested from KSRTC. |

The downloads were scripted on 2026-10-20 and can be repeated with `src/fetch_overture.py` and the git commands in `README.md`.

---

## A. Government transportation data

| # | Name | URL | Publisher | Format | Coverage | Fields available | Date | Licence | Level | Limitations | Proposed use |
|---|---|---|---|---|---|---|---|---|---|---|---|
| G1 | Divisionwise Route and route kms as on 31-03-2021 | https://www.data.gov.in/resource/divisionwise-route-and-route-kms-31-03-2021 | KSRTC, via the Open Government Data (OGD) Platform India | CSV, 584 bytes (`data/raw/manual/ksrtc_divisionwise_route_kms_2021-03-31.csv`, downloaded by the team on 2026-10-22) | KSRTC, 16 divisions | Sl No, Division, No of Routes, Route kms, Average route length. Totals: 5,471 routes and 426,382 route km, average 77.93 km. The division rows sum exactly to the corporation total. | Data as on 31 Mar 2021; published Jan 2022, updated Apr 2022 | Government Open Data Licence – India (GODL), the OGD default | **A** | Division totals only, with no route geometry or stops, and the data is five years old. The figures conflict with ksrtc.in "About us", which gives 13,273 routes and 9.49 lakh km on an undated basis. | Calibrate the size of the network and the mix of route lengths. Bengaluru Central division averages 243 km per route, so long intercity routes are concentrated there, which supports the Bengaluru-origin pilot. It cannot validate individual corridor distances. |
| G2 | KSRTC Operational results statistics (monthly) | https://www.data.gov.in/catalog/ksrtc-operational-results-statistics | KSRTC, via OGD India | CSV | KSRTC, by division | Physical and financial key indicators such as effective km, vehicle utilisation and revenue, by month | Multiple months (e.g. May 2022, Oct 2021) | GODL (to be confirmed) | B | Aggregated, with no route-level detail | Demand-side context, daily km per bus, and scenarios for fleet growth |
| G3 | Transport services offered by KSRTC (catalog) | https://www.data.gov.in/catalog/transport-services-offered-ksrtc | KSRTC, via OGD India | Catalog | KSRTC | The catalog contains G1. A search snippet reports that the catalog page showed "No Result Found". | n.d. | GODL | B | The catalog may be empty or moved | Entry point for G1 |
| G4 | KSRTC intercity e-bus programme (6 routes, 50 AC e-buses, FAME-II) | https://www.thenewsminute.com/article/ksrtc-operate-ac-electric-buses-six-intercity-routes-bengaluru-170608 | The News Minute; also Truck & Bus Builder, JMK Research | Web article | Bengaluru to Mysuru, Chikkamagaluru, Shivamogga, Madikeri, Virajpet and Davanagere | Routes, fleet size, a GCC rate of ₹55/km, and contract terms of 10 or 12 years (the reports disagree) | Nov 2021 – Jan 2023 | Copyright (cite only) | C | Press coverage, and the reports disagree on the operator and the contract term | **Default pilot corridors** for the model (see the audit report, §1) |
| G5 | List of bus depots in Karnataka | https://en.wikipedia.org/wiki/List_of_bus_depots_in_Karnataka | Wikipedia contributors | Web table | KSRTC, KKRTC, NWKRTC, BMTC | Depot names by division | Live page | CC BY-SA 4.0 | C (not opened: Wikipedia is blocked from the environment) | Crowd-sourced, with no coordinates | Cross-check the bus stations in the candidate sites (P2) against the named depots |
| G6 | KSRTC GTFS / timetable feed | — | — | — | — | — | — | — | **D** | No public GTFS feed for KSRTC intercity services was found. The unofficial `ksrtc-api` scraper (pkg.go.dev/github.com/rabilrbl/ksrtc-api) exists, but using it may breach the booking site's terms, so it is **not used**. | Ask KSRTC through faculty for route lists, schedules and depot locations |
| G7 | Public e-bus tenders: CESL PM e-Drive / PM e-Bus Sewa | https://www.electrive.com/2026/01/15/india-cesl-launches-tender-for-6230-electric-buses | CESL, as reported by electrive and others | Web article; the tender documents are on CESL/CPPP | National | Fleet volumes. Charging is bundled into the gross-cost contract bids, so no separate charger prices are published. | Dec 2025 – Jan 2026 | Copyright | C | No per-charger cost | Context for the procurement model |

## B. EV charging infrastructure

| # | Name | URL | Publisher | Format | Coverage | Fields available | Date | Licence | Level | Limitations | Proposed use |
|---|---|---|---|---|---|---|---|---|---|---|---|
| E1 | **Open Charge Map export, India folder** | https://github.com/openchargemap/ocm-export (folder `data/IN`, commit `8e3bedca…`) | Open Charge Map | JSON, one file per point of interest (POI); 1,872 India POIs | India; 379 of the POIs fall inside the Karnataka boundary | Title, address, latitude/longitude, operator, usage type, status, number of points, and per connection: type, current type, kW, amps, volts, quantity; also created, verified and status-update dates | Snapshot committed 2026-04-22 | CC BY 4.0 for all 379 Karnataka POIs (Data Provider "Open Charge Map Contributors") | **A** | Covers about 6% of the 6,097 public EVCS that the Ministry of Power reports for Karnataka (E3). It is car-oriented: only 1 site is ≥240 kW. | The existing public DC network, to assess whether highway sites could be shared and which places already have grid capacity |
| E2 | Overture Maps places: `ev_charging_station` | s3://overturemaps-us-west-2/release/2026-09-23.0/theme=places | Overture Maps Foundation | GeoParquet | Karnataka: 51 points | Name, confidence, source dataset | Release 2026-09-23.0 | CDLA-Permissive-2.0 (Overture places, per the Overture attribution docs; to be confirmed) | **A** | Sparse; all 51 come from the Microsoft source | Cross-check against E1 |
| E3 | Ministry of Power: state-wise public EVCS (PIB annexure) | https://static.pib.gov.in/WriteReadData/specificdocs/documents/2025/aug/doc202581598101.pdf | Ministry of Power, via PIB | PDF (`data/raw/manual/mop_public_evcs_statewise_pib_2025-08.pdf`, downloaded by the team on 2026-10-22), extracted to CSV by `src/extract_mop_evcs.py` | 36 states/UTs. National total 29,277; **Karnataka 6,097 (ranked first), confirmed in the file**. The states sum to the printed total. | PDF created 1 Aug 2025 | Public record (PIB) | **A** | Counts only, with no locations | The **coverage benchmark** for E1 (representativeness check) |
| E4 | Dataful: state/district/city-wise EV charging stations (sourced from MoP) | https://dataful.in/datasets/23369/ | Dataful (repackaging Ministry of Power data) | CSV (behind sign-in) | India | Charger type, rating, connector rating, number of connectors, coordinates | n.d. | Dataful terms (to be checked) | B | Not downloadable from the environment, and possibly behind a paywall | Optional enrichment if the team can download it |
| E5 | Kaggle: "Electric Vehicle Charging Stations in India" | https://www.kaggle.com/datasets/saketpradhan/electric-vehicle-charging-stations-in-india | Kaggle user saketpradhan | CSV | India | Station name, address, latitude/longitude (per a citing thesis) | n.d. | Unknown until the page is opened | B (URL taken from the NCI configuration manual https://norma.ncirl.ie/8633/2/abhijitsinghconfigurationmanual.pdf) | Kaggle is blocked from the environment, so the size, date and licence are unverified | Use only if its licence permits and it adds stations missing from E1 |
| E6 | Kaggle: "E-V charging stations" | https://www.kaggle.com/datasets/nezukokamaado/e-v-charging-stations | Kaggle user nezukokamaado | CSV | India | Per the same thesis | n.d. | Unknown | B | As for E5 | As for E5 |
| E7 | IEEE DataPort: EV charging station dataset | https://ieee-dataport.org/documents/electric-vehicle-charging-station-dataset | IEEE DataPort | — | Not verified | Not verified | n.d. | IEEE DataPort terms | B | Coverage unknown | Not planned |

## C. Geospatial data

| # | Name | URL | Publisher | Format | Coverage | Fields available | Date | Licence | Level | Limitations | Proposed use |
|---|---|---|---|---|---|---|---|---|---|---|---|
| S1 | **Overture Maps transportation segments (major roads)** | s3://overturemaps-us-west-2/release/2026-09-23.0/theme=transportation/type=segment | Overture Maps Foundation (derived mainly from OpenStreetMap) | GeoParquet | Karnataka bounding box; 116,389 segments intersect the state (motorway, trunk, primary, secondary) | Road class, name, route references (e.g. `IN:NH 48`), surface, speed limits, connectors (the junction topology), geometry | 2026-09-23.0 | ODbL 1.0 (OpenStreetMap-derived) | **A** | 60% of segments have no name, 51% no route reference, 90% no speed limit. One-way restrictions and lower-class roads are dropped. | The road graph for route distances, and for detour distances to candidate sites |
| S2 | **Overture Maps places: bus stations, fuel stations, rest stops** | s3://overturemaps-us-west-2/release/2026-09-23.0/theme=places | Overture Maps Foundation (sources: Meta, Microsoft, Foursquare, AllThePlaces) | GeoParquet | Karnataka: 2,115 bus stations, 3,291 fuel stations, 1 rest stop | Name, taxonomy, confidence, source dataset, address | 2026-09-23.0 | CDLA-Permissive-2.0 (to be confirmed) | **A** | Mixes KSRTC bus stands with private and city stops; 52 bus-station records have confidence below 0.5 | The **candidate charging-site set**: terminals and depots plus highway fuel stations |
| S3 | **Overture Maps divisions: state, districts, settlements** | s3://overturemaps-us-west-2/release/2026-09-23.0/theme=divisions | Overture Maps Foundation (OpenStreetMap-derived) | GeoParquet | Karnataka polygon, 31 district polygons, 24,807 settlements | Name, class (city, town, village, hamlet), population (only 329 records), Wikidata ID, geometry | 2026-09-23.0 | ODbL 1.0 | **A** | Population is sparse and often rounded. District names follow OSM and may differ from Census spellings. | Clip every layer to the state, label districts, and supply origin–destination towns |
| S4 | OpenStreetMap via Geofabrik or Overpass | https://download.geofabrik.de/asia/india/southern-zone.html | Geofabrik / OSM contributors | .osm.pbf | Southern India | Full OSM tags, including bus-route relations | Daily | ODbL 1.0 | **B (blocked)** | Unreachable from the environment, so S1–S3 (built from OSM) stand in for it | Optional later: OSM bus-route relations and depot tags |
| S5 | Datameet maps (India administrative boundaries) | https://github.com/datameet/maps | DataMeet community | Shapefile/GeoJSON | India | Boundaries | Various | See the repository licence (not checked) | Reachable, not downloaded | S3 already supplies the boundaries | Fallback if Census district codes are needed |

## D. Electric bus specifications (summary; values are in `data/raw/manual/parameters_register.csv`)

| # | Item | Best available value | Source | Level |
|---|---|---|---|---|
| B1 | Battery capacity, 12 m intercity coach | 360 kWh (Olectra C9 at launch); 422 kWh (later listing) | Sustainable Bus 2020; ev.care | C |
| B2 | Energy consumption | 1.3 kWh/km (tender allowance, AIS-039, AC off); range 1.2–1.5 | Chennai MTC tender, pre-bid replies (TNIDB, 15 Jun 2024); Kerala RTC tender coverage | B / C |
| B3 | Maximum charging power | 240 kW CCS2 (C9); MoP sets a 240 kW minimum for e-bus depots | ev.care; MoP EVCI Guidelines 2024 | C / B |
| B4 | Usable battery % and minimum SoC reserve | **Not found** | — | D |
| B5 | Charging curve | **Not found**. Only "0–80% in about 2.5 h at 240 kW" | ev.care | C / D |
| B6 | Claimed range | 250–300 km (Karnataka programme), 350 km (third party) | OEM and press | C |

## E. Financial data (summary; values are in the parameters register)

| # | Item | Best available value | Source | Level |
|---|---|---|---|---|
| F1 | EV charging energy tariff, Karnataka | **₹4.50/kWh (LT-6(c) and HT), FY2025-26 to FY2027-28, confirmed** (Annexure-9 p.542). KSRTC depots on HT supply get a time-of-day adjustment from December to June: +₹1 at 06–09h and 18–22h, −₹1 at 22–06h (so overnight charging costs ₹3.50). There is no adjustment from July to November. | KERC Combined Tariff Order 2025, 27 Mar 2025 (text in `data/raw/manual/kerc_combined_tariff_order_2025_text.txt`, supplied by the team on 2026-10-22) | **A** |
| F2 | Fixed and demand charges | **₹70/kW/month (LT); ₹200/kVA/month (HT), confirmed** (Annexure-9 p.542). HT billing demand is the higher of the recorded maximum demand and 90% of contract demand (p.545). | KERC Combined Tariff Order 2025 | **A** |
| F3 | Charger installation cost | **No Indian per-unit price.** Benchmarks: ₹2.29 cr for 15 charging points (Amritsar); ₹31 cr for 9 depots (Delhi DTC) | The Tribune | C |
| F4 | Grid connection cost | Upper benchmark ₹300.61 cr for 7 depots, including 66 kV in-feeds | ThePrint/PTI | C |
| F5 | Site cost | ₹1/kWh revenue share on public land (MoP 2024). Rental on private highway sites: **not found** | Mercom (reporting MoP) | C / D |
| F6 | Operations and maintenance | **Not found** | — | D |

## Requests to raise at the D2 consultation (Wed 21 Oct)

1. Route-level route km, schedules (trips per day) and depot coordinates for the six pilot corridors (G6). G1 gives division totals only.
2. Vehicle specification of the e-buses in service: battery kWh, SoC operating window, observed kWh/km.
3. Any KSRTC or CESL bill of quantities (BOQ) for depot chargers, including the transformer and civil works.
