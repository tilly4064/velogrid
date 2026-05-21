# VeloGrid

**A Brisbane cycling infrastructure explorer.**

VeloGrid is a Brisbane cycling infrastructure explorer that layers official bikeways, cyclist crash history, road hierarchy, parks, and bike racks on one interactive map. Live QLD traffic alerts and OpenRouteService routing included. All powered by public open data. No accounts, no tracking. Just the map and what's on it.

**Live map: https://tilly4064.github.io/velogrid/**

Best viewed on desktop. The sidebar is 340 px wide; mobile is functional but cramped.

---

## What it does

VeloGrid brings several public datasets onto one map so planners and curious riders can see where the network is, where it isn't, and where riders feel safe or unsafe.

**Map layers** (toggle on/off; only bikeways shown by default to keep first load clean):

- **Bikeway sections** — every official bikeway section in the Brisbane LGA, coloured by traffic type (separated pathway, bicycle lane, shared path, and so on), with attributes on click.
- **Road hierarchy** — the City Plan 2014 classification for every road segment, coloured from motorway (red) through neighbourhood/local (green).
- **Cyclist crashes** — recorded cyclist crashes in the Brisbane LGA from 2001 to mid-2024, shown as a density heatmap at low zoom and individual severity-coded points at high zoom.
- **Safe & unsafe spots** — crowdsourced cyclist perceptions from the 2023 BikeSpot survey. Green pins are spots riders felt safe; red pins are spots they flagged as unsafe. Pin size scales with how many people endorsed the observation.
- **Bicycle racks** — public bike parking infrastructure, shown as triangle markers to stand apart from the circular point layers.
- **Parks** — council-managed park polygons across the LGA.

**Tools:**

- **Route planner** — click to set a start and destination, then compare a "Fastest" route against a "Most cycling friendly" route side-by-side, with distance, time, and elevation gain for each. Markers are draggable; routes recompute on drag.
- **Traffic updates for cyclists** — live QLDTraffic events filtered for likely cyclist relevance, refreshed every five minutes. Click an event to highlight the affected road segment on the map.

Click almost anything on the map for details. The legend (top-right) groups the multi-colour layers and starts collapsed.

## Data sources

All data is loaded live from public APIs at page load, except the BikeSpot survey, which is bundled as a pre-processed local file because no API exists for it.

| Layer | Source | Format |
|---|---|---|
| Bikeways | Brisbane City Council ArcGIS FeatureServer | GeoJSON |
| Cyclist crashes | QLD Government CKAN DataStore (resource `e88943c0…`) | JSON, points |
| Road hierarchy | BCC Opendatasoft (City Plan 2014 overlay) | GeoJSON |
| Parks | BCC Opendatasoft (Park Locations) | GeoJSON |
| Bicycle racks | BCC Opendatasoft (Bicycle Rack locations) | GeoJSON |
| Suburb boundaries | BCC Opendatasoft (used to filter survey points to Brisbane suburbs) | GeoJSON |
| Safe & unsafe spots | BikeSpot 2023 survey (pre-processed local file) | GeoJSON |
| LGA boundary | QLD Spatial Information (Admin Boundaries Framework) | GeoJSON |
| Live traffic | QLDTraffic API v2 (via corsproxy.io) | GeoJSON |
| Routing | OpenRouteService Directions v2 (cycling-regular) | GeoJSON |
| Map tiles | CARTO Dark Matter | Raster |

## Stack

- **MapLibre GL JS 4.7.1** for map rendering (loaded from jsDelivr CDN)
- **Vanilla JavaScript** — no framework, no build step, no npm dependencies
- **`sessionStorage`** for client-side caching of static datasets between page reloads
- **`fetch` with `AbortController`** for timed-out API requests
- **Two files**: `index.html` (the app) and `bikespot_brisbane.geojson` (the survey data), served same-origin

## Limitations (read these)

These are real constraints of the data, not bugs:

- **The crash heatmap shows where crashes were recorded, not where risk is highest.** Roads with no recorded crashes may simply be roads no one cycles on. Treat density as the numerator of risk, not risk itself.
- **The road hierarchy reflects City Plan 2014.** Updates to the planning instrument are infrequent. Some "future" categories may have been built; some classifications may have changed in the decade since.
- **Crash data ends in mid-2024.** The QLD dataset is published with significant lag for verification, so recent crashes will not appear.
- **Safe & unsafe spots are perceived, not measured.** The 2023 BikeSpot survey was voluntary and self-selected. Unsafe spots vastly outnumber safe ones because participants were more motivated to flag problems than to praise good ones, and pin size reflects how many participants agreed — community sentiment, not a risk model.
- **Routes come from OpenRouteService using OpenStreetMap data; bikeways come from BCC.** The two sources will sometimes disagree on whether a road carries cycling infrastructure. Both layers are shown so users can judge.
- **"Most cycling friendly" is ORS's recommended cycling preference** — it biases toward OSM-tagged cycling infrastructure. It is not a safety rating and makes no claim about lowest stress.
- **Traffic events are filtered for cyclist relevance by heuristic, not by an authoritative field.** QLDTraffic does not tag events by cyclist impact, so the relevance score is interpretive.
- **Bicycle rack coverage reflects what BCC has audited**, not every rack in Brisbane.
- **VeloGrid is a research and visualisation tool, not a navigation app.** Verify conditions locally before relying on it for a real ride.

## Running your own copy

Two-file static deployment — works on GitHub Pages, Netlify, Cloudflare Pages, or any static host.

1. Generate a free [OpenRouteService API key](https://openrouteservice.org/dev/#/signup) (2,000 requests/day, no credit card).
2. In `index.html`, find `ORS_API_KEY = "..."` and paste your key.
3. If your host has a fixed domain, restrict the ORS key by HTTP referrer to that domain in the ORS dashboard where available.
4. Upload `index.html` and `bikespot_brisbane.geojson` to the repository root (same folder, exact filenames).
5. On GitHub Pages: Settings → Pages → Deploy from a branch → `main` / root.

The QLDTraffic public API key is included in the source. It is shared and rate-limited globally — fine for a low-traffic deployment.

## Architecture notes

- **All data loaders run in parallel** via `Promise.allSettled`, so one failure never blocks the others.
- **Caching is versioned** — bumping `CACHE_VERSION` invalidates all prior `sessionStorage` entries, and stale entries from older versions are purged on load.
- **MapLibre style expressions** colour features lazily, avoiding per-feature property mutation.
- **Survey points are filtered to Brisbane suburbs in-browser** using point-in-polygon against the live BCC suburb boundaries, with the Moreton Bay island localities excluded.
- **Routes are fetched in parallel** and rendered in distinct colours; near-identical routes (common in suburbs with one practical path) are flagged with a note.
- **Traffic events are filtered to the LGA by point-in-polygon**, not a bounding box, so events in adjacent councils are correctly excluded.

## Licence

Code released under the MIT licence. Data is owned by its respective publishers — see each source link above for licensing.

## Acknowledgements

Built with public open data from Brisbane City Council, the Queensland Department of Transport and Main Roads, the BikeSpot 2023 survey, and OpenStreetMap contributors (via OpenRouteService and CARTO basemap tiles). Map rendering by MapLibre GL JS.
