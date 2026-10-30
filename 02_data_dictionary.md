# Data dictionary: data-v1.2

> v1.2 equals v1.1 plus `ksrtc_divisions_route_km_2021.csv` and `mop_public_evcs_statewise_2025-08.csv`, with an updated `parameters_register.csv` (verified KERC tariffs, MoP count).

> v1.1 equals v1.0 plus the `bus_facility_class` column in `candidate_sites_overture.csv` and the new file `road_graph_nodes.csv.gz`. Every other table is unchanged.

All files are in `data/processed/v1.2/` and are produced by `src/clean.py`. `CHECKSUMS.sha256` lists their SHA-256 hashes. The coordinate system is WGS84 (EPSG:4326), with longitude and latitude in decimal degrees. "Derived" means the team computed the field, and it is listed again in the feature engineering log (audit report §5).

## ev_stations_ocm_karnataka.csv: existing public EV chargers (379 rows, one per site)

| Field | Type | Description | Source / derivation |
|---|---|---|---|
| ocm_id | int | Open Charge Map POI ID, the primary key | OCM `ID` |
| title | str | Site name | `AddressInfo.Title` |
| town, state_raw, postcode | str | Address fields as entered (free text) | `AddressInfo` |
| lat, lon | float | Coordinates | `AddressInfo.Latitude/Longitude` |
| operator | str | Charge point operator (e.g. Tata Power, Statiq) | `OperatorID` → referencedata |
| usage_type | str | Public / membership / private | `UsageTypeID` → referencedata |
| status | str | Operational / Temporarily Unavailable / Not Operational | `StatusTypeID` → referencedata |
| number_of_points | int | Number of vehicles that can charge at once | `NumberOfPoints` |
| n_connections | int | Connection records listed for the site | derived (count) |
| max_power_kw | float | Highest connector power at the site, kW | derived (max of `Connections.PowerKW`) |
| has_dc | bool | At least one DC connector | derived (`CurrentTypeID` = 30) |
| has_ccs2 | bool | At least one CCS (Type 2) connector, the standard used by Indian e-buses | derived (`ConnectionTypeID` = 33) |
| bus_capable_dc_ge_60kw | bool | DC and ≥ 60 kW. This is a screening flag only: a 12 m bus needs about 240 kW to recharge within a layover. | derived |
| data_provider, licence | str | Who supplied the record, and its licence | `DataProviderID` → referencedata |
| date_created, date_last_verified, date_last_status_update | ISO datetime | Record timestamps | OCM |
| district | str | District containing the point | derived (point-in-polygon on districts) |
| power_outlier | bool | max_power_kw > 500 | derived (quality flag) |
| near_duplicate_50m | bool | Lies within 50 m of an earlier OCM record | derived (quality flag) |

## ev_connections_ocm_karnataka.csv: connectors (585 rows)

`ocm_id` (a foreign key to the file above), `connection_type`, `current_type`, `power_kw`, `amps`, `voltage`, `quantity`, `status`. All of these come from OCM `Connections[]` after lookup in referencedata.

## candidate_sites_overture.csv: candidate charging sites (5,458 rows)

| Field | Type | Description |
|---|---|---|
| place_id | str | Overture place GERS ID, the primary key |
| name | str | Place name |
| site_type | str | `bus_station`, `fuel_station`, `ev_charging_station` or `rest_stop` (derived from the taxonomy) |
| taxonomy_primary | str | Overture taxonomy category as published |
| confidence | float 0–1 | Overture's confidence that the place exists |
| low_confidence | bool | confidence < 0.5 (derived flag) |
| operating_status | str | Overture operating status (empty in this release for every row) |
| bus_facility_class | str | For bus stations only (derived from the name, v1.1): `terminal_or_depot` (404), `bus_stop` (1,589), `unclassified` (90), `likely_misclassified` (17, e.g. ATMs and pharmacies), `private_operator_office` (13), `metro_station` (2) |
| near_duplicate_50m | bool | Within 50 m of a higher-confidence place of the same type (derived flag) |
| locality | str | Address locality |
| district | str | District, by point-in-polygon (derived) |
| source_datasets | str | Upstream sources merged by Overture (Meta, Microsoft, Foursquare, AllThePlaces) |
| lon, lat | float | Coordinates |

## road_segments_major.csv.gz: road network attributes (116,389 rows)

| Field | Type | Description |
|---|---|---|
| segment_id | str | Overture segment ID |
| name | str | Road name (missing for 60%) |
| road_class | str | motorway / trunk / primary / secondary |
| subclass | str | e.g. link |
| route_refs | str | Semicolon-separated network and reference, e.g. `IN:NH 48` (derived from `routes`) |
| surface | str | First surface value on the segment (derived) |
| max_speed_kmh | float | Highest posted maximum speed on the segment (derived; 90% missing) |
| length_m | float | Geodesic length in metres (derived; haversine over the vertices) |
| n_connectors | int | Junction points on the segment |
| in_giant_component | bool | The segment touches the largest connected road component (derived) |
| wkt | str | LineString geometry as WKT, 6 decimal places |

## road_graph_edges.csv.gz: routable graph (169,153 edges, 161,526 nodes)

`from_connector`, `to_connector` (Overture connector IDs), `length_m` (the segment length times the fraction between the two connectors), `segment_id`, `road_class`. The graph is **undirected**, has no one-way or turn restrictions, and includes only motorway, trunk, primary and secondary roads.

## road_graph_nodes.csv.gz: graph node coordinates (v1.1)

`connector_id`, `lon`, `lat`. Only nodes in the giant component are included.

## towns.csv: settlements (24,807 rows)

`town_id` (Overture division ID), `name`, `settlement_class` (city / town / village / hamlet), `population` (only 329 non-null), `district`, `wikidata`, `lon`, `lat`.

## districts.csv (31 rows)

`district_id`, `district_name`, `area_km2_approx` (derived with a planar degree-to-km conversion, accurate to about ±1%), `centroid_lon`, `centroid_lat`.

## karnataka_boundary_wkt.csv (1 row)

`name`, `wkt`: the state polygon used to clip every layer.

## routing_smoke_test.csv (9 rows)

`from`, `to`, `straight_line_km`, `network_km` (shortest path on the graph), `circuity` (network ÷ straight-line distance), `snap_from_m`, `snap_to_m` (the distance from the town point to the nearest graph node).

## parameters_register.csv (25 rows): bus, tariff, cost and operations parameters

| Field | Description |
|---|---|
| param_id | Key: BUS, CHG, TAR, LAND, CAPEX, OPEX or OPS, followed by a number |
| category, parameter, applies_to | What the value is, and for which vehicle, site or region |
| value_low, value_central, value_high | Numeric value or range. Blank means no verified value exists. |
| unit | Unit of measure |
| source_title, source_url, publisher, source_date | Citation |
| verification_status | `official_doc_not_opened`, `secondary_report`, `unverified_secondary`, `gap`, or `gap_assumption_needed` (see the source register) |
| notes | Conflicts, caveats and what to verify |

## quality_profile.json

A machine-readable output of the quality checks: row counts, missing values per field, duplicates, outliers, licence counts, graph connectivity and the routing test. The audit report quotes from this file.

## ksrtc_divisions_route_km_2021.csv (16 rows; source G1, data as on 31 Mar 2021)

`sl_no`, `division`, `n_routes`, `route_km`, `avg_route_km` (as published, = route_km ÷ n_routes). The corporation total row is removed and checked: the division rows sum to 5,471 routes and 426,382 km.

## mop_public_evcs_statewise_2025-08.csv (36 rows; source E3)

`s_no`, `state_ut`, `public_charging_stations`. Sums to 29,277; Karnataka has 6,097.

## Phase Two outputs: data/processed/corridors_v1.2/ (from src/corridors.py)

- **corridor_summary.csv**, one row per corridor: endpoints and how each was chosen, snap distances, `network_km`, `straight_line_km`, `circuity`, km by road class, share of km on national highways, main route references, `longest_single_edge_km`, intermediate terminals with km from origin, `max_gap_between_terminals_km`, counts of public chargers within 1 km (all / CCS2 DC ≥60 kW / CCS2 ≥120 kW), `max_gap_between_ge_120kw_chargers_km`, fuel stations within 200 m, `trip_frequency_per_day` (empty: no data), and `one_way_kwh_low/central/high` at 1.2/1.3/1.5 kWh/km.
- **corridor_intermediate_terminals.csv**, **corridor_existing_chargers.csv**, **corridor_fuel_stations.csv**: the matched points, with `chainage_km` (network km from origin at the nearest route vertex) and `dist_to_route_m`.
- **corridor_routes.geojson**: route lines (WGS84), drawn through the graph nodes.
