# Substation Proximity

A QGIS plugin for locating and costing power substations near a point of interest.

Click anywhere on the map and the plugin draws dashed lines to every substation within your chosen radius, labeling each with its name, voltage level, straight-line distance, and estimated cable route cost for both 20 kV and 110 kV connections. Built for data center siting analysis but useful for any proximity assessment involving electrical infrastructure.

---

## Features

- **Click-to-query** — click anywhere on the map canvas; the plugin immediately draws lines to all nearby substations
- **Voltage filtering** — filter results to show only 110 kV, 220 kV, 380 kV substations, or any combination; useful for ignoring medium-voltage distribution equipment ("trafo houses") that is typically irrelevant for large loads
- **Cost estimation** — each label shows estimated cable route costs for both 20 kV and 110 kV connections, calculated from straight-line distance × a configurable deviation factor × a configurable cost-per-km rate
- **Configurable parameters** — deviation factor and cost-per-km for each voltage tier are editable in a settings dialog, allowing you to update for inflation or adjust to local unit costs
- **Cached results** — substation data is fetched once per area and held in memory; subsequent clicks within the same area are instant

---

## Cost Estimation

For each substation the label shows:

```
UW Rödelheim         3.8 km
7.4 M€ / 20 kV
11.9 M€ / 110 kV
```

The formula is:

```
Cost = distance × deviation_factor × cost_per_km
```

**Default parameters (all editable):**

| Parameter | Default |
|---|---|
| Deviation factor | 1.3 |
| 20 kV cost per km | 1.5 M€ |
| 110 kV cost per km | 2.4 M€ |

The deviation factor accounts for the fact that cable routes cannot follow straight lines — they follow roads, rights-of-way, and other constraints. A factor of 1.3 means the routed cable length is assumed to be 30% longer than the straight-line distance.

To change any parameter: click the **settings (⚙)** button in the plugin toolbar.

---

## Data Source

Substation data comes from **OpenStreetMap** via the **Overpass API**. The plugin queries for all features tagged `power=substation`, including both node-based and way-based (polygon) substations. Way-based substations are represented by their centroid.

Four Overpass endpoints are tried in sequence, falling back to mirrors if the primary server is unavailable:

- `overpass-api.de` (primary)
- `overpass.kumi.systems`
- `maps.mail.ru`
- `overpass.openstreetmap.ru`

Coverage and accuracy depend on OSM community activity. Germany and Western Europe are well covered. Substation names are only as complete as what contributors have entered — unnamed substations display distance and cost only.

---

## Requirements

The project CRS must use **metres** as its unit. The plugin will not produce correct distances in a geographic CRS (e.g. EPSG:4326) because those use degrees.

To set the project CRS: **Project → Properties → CRS** (or `Ctrl+Shift+P`), type your EPSG code in the filter box, and click OK.

**Recommended CRS by region:**

| Region | CRS |
|---|---|
| Germany / Frankfurt | EPSG:25832 (UTM zone 32N) |
| UK | EPSG:27700 (British National Grid) |
| Toronto / Ontario | EPSG:32617 (UTM zone 17N) |

To find the right UTM zone for any location: take the longitude, add 180, divide by 6, round up. The EPSG code is `326XX` for the northern hemisphere and `327XX` for the southern, where XX is the zone number.

You can confirm you're in a metric CRS by checking the coordinate display at the bottom-right of the QGIS canvas. Six- or seven-digit numbers (e.g. `477000, 5552000`) indicate metres. Small decimals (e.g. `8.68, 50.11`) mean you're still in degrees and need to switch.

---

## Loading Times

**First click:** 2–5 seconds. The plugin fetches all substations within the current canvas extent plus a buffer zone from Overpass and caches them in memory.

**Subsequent clicks within the cached area:** Instant. No network request is made.

**Clicking outside the cached area:** 2–5 seconds for that click while new data is fetched and merged into the cache. Subsequent clicks in the newly fetched area are instant.

**If all Overpass mirrors are unreachable:** The click fails after attempting all four endpoints (~30 s total timeout). A message is printed to the QGIS Python console; try again once connectivity is restored.

---

## Caching Behaviour

The cache is cumulative and persists for the duration of the QGIS session. As you click across different areas, the cache grows and is never pruned. Voltage filter changes reuse the existing cache — no re-fetch is needed. Data is lost when QGIS is closed or the plugin is unloaded.

---

## Installation

**Plugins → Manage and Install Plugins → Install from ZIP → select `substation_proximity.zip`**

---

## Roadmap

- **Substation upgrade plans database** — pop-up info boxes showing known grid reinforcement and upgrade timelines from DSO planning data, to support faster site reviews without manual lookups

---

## Notes

- Results are straight-line distances; actual cable route lengths will be longer (hence the deviation factor)
- Voltage levels are taken from OSM tags and are only as accurate as what has been entered by contributors; always verify against official DSO data before use
- The plugin is designed for the transmission voltage levels relevant to large loads (110 kV and above); the voltage filter is recommended to reduce clutter from medium-voltage distribution equipment
