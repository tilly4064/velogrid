# VeloGrid
A visual support tool for cycling infrastructure planning in Brisbane.

VeloGrid layers Brisbane's cycling infrastructure datasets onto one interactive map — bikeways, cyclist crash history, road hierarchy, parks, bike racks, crowdsourced safety perceptions, electric micromobility trip patterns, and Census cycling-to-work data. By making these datasets visible together, it helps planners, researchers, and curious riders understand where the network is strong, where gaps exist, and where investment could have the greatest impact on safe cycling expansion.

Live map: **https://tilly4064.github.io/velogrid/**

Best viewed on desktop. The sidebar is 340 px wide; mobile is functional but cramped.

---

## What it does

VeloGrid brings several public datasets onto one map so planners and curious riders can see where the network is, where it isn't, and where riders feel safe or unsafe.

### Map layers
Toggle on/off. Only bikeway sections are shown by default.

| Layer | What it shows |
|---|---|
| **Bikeway sections** | Every official BCC bikeway, coloured by traffic type (separated, shared path, painted lane, etc.). Click for attributes. |
| **Road hierarchy** | City Plan 2014 classification for every road segment, from motorway (red) to local street (green). |
| **Cyclist crashes** | Recorded crashes 2001–mid-2024, as a density heatmap at low zoom and severity-coded points at high zoom. |
| **Safe & unsafe spots** | Crowdsourced perceptions from the 2023 BikeSpot survey. Pin size scales with community endorsement. |
| **Bicycle racks** | Public bike parking. |
| **Parks** | Council-managed park polygons across the LGA. |
| **Cycling to work (Census 2021)** | SA2 choropleth of journey-to-work cycling mode share, pre-computed from ABS Census data. |
| **Electric micromobility trips** | Trip-count heatmap of e-scooter and e-bike routes. Colour and line width scale logarithmically with trip count. |

### Tools
- **Route planner** — set a start and destination and get a cycling route (OpenRouteService, cycling-regular profile). Markers are draggable.
- **Traffic updates for cyclists** — live QLDTraffic events filtered for likely cyclist relevance, refreshed every five minutes. Click to highlight the affected road segment.

---

## Data sources

All data loads live from public APIs at page load, except BikeSpot (bundled local file — no public API exists for it) and the two pre-processed files below.

| Layer | Source |
|---|---|
| Bikeways | Brisbane City Council ArcGIS FeatureServer |
| Cyclist crashes | QLD Government CKAN DataStore (resource `e88943c0…`) |
| Road hierarchy | BCC Opendatasoft (City Plan 2014 overlay) |
| Parks | BCC Opendatasoft (Park Locations) |
| Bicycle racks | BCC Opendatasoft (Bicycle Rack locations) |
| Suburb boundaries | BCC Opendatasoft (used to filter survey points to Brisbane) |
| Safe & unsafe spots | BikeSpot 2023 survey — bundled pre-processed local file |
| Cycling to work | ABS Census 2021, SDMX Data API (C21_G62_SA2) joined to ASGS SA2 boundaries — bundled pre-processed local file |
| Electric micromobility | Ride Report Micromobility Index |
| LGA boundary | QLD Spatial Information (Admin Boundaries Framework) |
| Live traffic | QLDTraffic API v2 (via corsproxy.io) |
| Routing | OpenRouteService Directions v2 (cycling-regular) |
| Map tiles | CARTO Positron (light) |

---

## Stack

- **MapLibre GL JS 4.7.1** for map rendering (jsDelivr CDN)
- **Vanilla JavaScript** — no framework, no build step, no npm dependencies
- `sessionStorage` for client-side caching of static datasets
- `fetch` with `AbortController` for timed-out API requests
- Three static files: `index.html`, `bikespot_brisbane.geojson`, `cycling_to_work_brisbane.geojson`, and `micromobility_brisbane.geojson`

---

## Limitations

- The crash heatmap shows where crashes were recorded, not where risk is highest. Low-density roads may simply be roads no one cycles on — treat density as the numerator of risk, not risk itself.
- Road hierarchy reflects City Plan 2014. Classifications may not match current conditions.
- Crash data ends mid-2024 due to QLD publication lag.
- BikeSpot spots are perceived safety, not measured risk. Self-selected participation means unsafe spots vastly outnumber safe ones.
- Routes come from OpenRouteService (OSM data); bikeways come from BCC. The two sources will sometimes disagree.
- Cycling-to-work data is journey-to-work only, residence-based, and reflects 2021 Census conditions (COVID-affected work-from-home patterns). ABS random perturbation affects small cell counts.
- Micromobility data covers all vehicle types; it cannot be disaggregated by mode within the app.
- Traffic event relevance is scored by heuristic — QLDTraffic does not tag events by cyclist impact.
- Bicycle rack coverage reflects BCC audits, not every rack in Brisbane.

VeloGrid is a research and visualisation tool, not a navigation app. Verify conditions locally before relying on it for a real ride.

---

## Running your own copy

Two-file static deployment — works on GitHub Pages, Netlify, Cloudflare Pages, or any static host.

1. Generate a free [OpenRouteService API key](https://openrouteservice.org/) (2,000 requests/day, no credit card).
2. In `index.html`, find `ORS_API_KEY = "..."` and paste your key.
3. If your host has a fixed domain, restrict the ORS key by HTTP referrer in the ORS dashboard.
4. Upload `index.html`, `bikespot_brisbane.geojson`, `cycling_to_work_brisbane.geojson`, and `micromobility_brisbane.geojson` to the repository root (same folder, exact filenames).
5. On GitHub Pages: **Settings → Pages → Deploy from a branch → main / root**.

The QLDTraffic public API key is included in the source. It is shared and rate-limited globally — fine for a low-traffic deployment.

---

## Architecture notes

- All data loaders run in parallel via `Promise.allSettled` — one failure never blocks the others.
- Caching is versioned: bumping `CACHE_VERSION` invalidates all prior `sessionStorage` entries; stale entries are purged on load.
- MapLibre style expressions colour features lazily, avoiding per-feature property mutation.
- Survey points are filtered to Brisbane suburbs in-browser via point-in-polygon against live BCC suburb boundaries.
- Micromobility colours and line widths use a log-scale interpolation over the `count` property (range 100–972,400 trips).
- Traffic events are filtered to the LGA by point-in-polygon, not bounding box.

---

## Licence

Code released under the MIT licence. Data is owned by its respective publishers — see each source link above for licensing.

## Acknowledgements

Built with public open data from Brisbane City Council, the Queensland Department of Transport and Main Roads, the Australian Bureau of Statistics, the BikeSpot 2023 survey, and OpenStreetMap contributors (via OpenRouteService and CARTO basemap tiles). Map rendering by MapLibre GL JS.
