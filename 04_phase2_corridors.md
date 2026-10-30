---
title: "Phase Two: pilot corridor profiles"
data: "data/processed/corridors_v1.2/ (built by src/corridors.py from data-v1.1)"
date: "2026-10-20"
---

# Phase Two: pilot corridor profiles

## Why these corridors

KSRTC route-level data (schedules, stops, route km) is not openly downloadable. The six routes on which KSRTC already runs 12 m AC e-buses from Bengaluru (source G4) are therefore the most defensible pilot. They are real services, they cover three distinct highway axes (NH 275 to the south-west, NH 48 to the north-west, and NH 73/69 to the west), and they span the range-critical distance band of 140–275 km.

All routes start at Kempegowda (Majestic) bus station. KSRTC also uses the Mysuru Road satellite station for southern routes; that origin is a scenario for later phases.

## Results (computed, not assumed)

| Corridor | Network km | Straight-line km | Circuity | Main highways | Terminals on route (≤500 m) | Longest gap between terminals (km) | Public chargers ≤1 km (all / CCS2 DC ≥60 kW / CCS2 ≥120 kW) | Longest gap without a CCS2 ≥120 kW charger (km) | One-way energy at 1.2 / 1.3 / 1.5 kWh/km |
|---|---|---|---|---|---|---|---|---|---|
| C1 Mysuru | 137.5 | 123.7 | 1.11 | NH 275 | 4 | 42.0 | 20 / 11 / 1 | 131.0 | 165 / 179 / 206 kWh |
| C2 Chikkamagaluru | 229.7 | 198.4 | 1.16 | NH 48, NH 73 | 4 | 72.1 | 23 / 12 / 5 | 173.0 | 276 / 299 / 345 kWh |
| C3 Shivamogga | 273.6 | 241.2 | 1.13 | NH 48, NH 73, NH 69 | 4 | 106.8 | 21 / 11 / 5 | 216.9 | 328 / 356 / 410 kWh |
| C4 Madikeri | 253.0 | 208.4 | 1.21 | NH 275 | 8 | 57.6 | 31 / 18 / 1 | 246.5 | 304 / 329 / 380 kWh |
| C5 Virajpet | 240.5 | 210.5 | 1.14 | NH 275, SH 90 | 6 | 61.8 | 29 / 16 / 1 | 234.0 | 289 / 313 / 361 kWh |
| C6 Davanagere | 258.0 | 243.4 | 1.06 | NH 48 | 3 | 151.0 | 27 / 15 / 7 | 71.2 | 310 / 335 / 387 kWh |

Detailed lists are in `corridor_intermediate_terminals.csv` (stops with their km from origin), `corridor_existing_chargers.csv`, and `corridor_fuel_stations.csv` (36–74 roadside fuel stations per corridor, the en-route candidate pool). Route lines are in `corridor_routes.geojson`.

**Trip frequency:** not available for any corridor. The data request has gone to KSRTC, and Phase Three will use demand scenarios.

## What this already tells us

1. **Only Mysuru fits comfortably in one charge.** At the central consumption of 1.3 kWh/km, the five longer corridors need 299–356 kWh one way, against a 360 kWh nameplate battery. Usable energy above reserve will be lower than that (Phase Three will quantify it), so these trips need **destination or en-route charging**, and possibly both under high consumption. This is an energy calculation on measured distances, not yet a feasibility verdict.
2. **Existing public chargers do not fill the gap.** CCS2 chargers of 120 kW or more cluster within 60 km of Bengaluru. Beyond that, the Mysuru, Madikeri and Virajpet corridors have none for 131–247 km. Only the NH 48 (Davanagere) corridor has several. Almost all of these are car-oriented sites at hotels and restaurants, so Baseline 1 will perform poorly by construction, and the report should say so.
3. **Highway bypasses push terminals off the route.** On NH 48, the Tumakuru, Sira and Chitradurga bus stands lie more than 500 m from the bypass, so the Davanagere corridor has a 151 km stretch with no terminal on the route. En-route charging there must be either at a roadside site or at a terminal reached by a detour. Phase Four will measure that detour on the network.
4. **Natural intermediate hubs appear on several corridors:** Ramanagara, Maddur and Mandya (C1, C4 and C5 share the NH 275 trunk for about 100 km), Hunsur (C4, C5), Tumakuru, Tiptur and Arsikere (C2, C3). Sites on shared trunks can serve several routes, which is exactly what Model A rewards.

## Validation status

- **Distances.** Circuity of 1.06–1.21 is plausible for highways. Every endpoint snaps to the graph within 233 m. KSRTC's published route km (G1, now downloaded) are division totals, so they cannot check individual corridors. They do support the pilot choice: Bengaluru Central division averages 243 km per route, against a corporation average of 77.9 km. Corridor distances therefore remain modelled network distances, not measured operating km, until route-level km are obtained from KSRTC.
- **Route choice.** The shortest path is not necessarily KSRTC's operated path. Chikkamagaluru runs via Tiptur and Javagal on the network, while KSRTC may run via Hassan. The corridor definition allows a "via" town, so this can be corrected once real routes are known.
- **Stops.** "Terminals on route" are bus stands or depots in Overture within 500 m of the path. They are a proxy for intermediate stops, not KSRTC's stopping pattern.
- **Chargers.** A 1 km straight-line screen is used here. Network detour and bus compatibility (bay size, access) will be checked in Phase Four.

## How this scales

Every step is parameterised by an origin, a destination and a road graph that already covers the whole state. Adding a corridor means adding one line to `CORRIDORS` in `src/corridors.py`. A statewide model needs only the KSRTC route list (origin, destination, via, trips per day). The coverage sets and models in Phase Five are built per route, so their size grows with the number of routes, not with the area of the state.
