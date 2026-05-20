# VeloGrid

**A Brisbane cycling infrastructure explorer.**

VeloGrid is a Brisbane cycling infrastructure explorer that layers official bikeways, cyclist crash history, road hierarchy, parks, and bike racks on one interactive map. Live QLD traffic alerts and OpenRouteService routing included. All powered by public open data. No accounts, no tracking. Just the map and what's on it.

---

## What it does

- **Bikeways** — every official bikeway section in the Brisbane LGA, coloured by traffic type (separated pathway, bicycle lane, shared path, etc.), with attributes on click.
- **Cyclist crashes** — every recorded cyclist crash in Brisbane LGA from 2001 to mid-2024, rendered as a kernel-density heatmap at low zoom and individual severity-coded points at high zoom.
- **Road hierarchy** — the City Plan 2014 classification for every road segment, coloured from motorway (red) through neighbourhood/local (green).
- **Parks** — council-managed park polygons across the LGA.
- **Bicycle racks** — public bike parking infrastructure with capacity figures.
- **Live traffic alerts** — QLDTraffic events in Brisbane, filtered for cyclist relevance, refreshed every 5 minutes. Click an event to highlight the affected road segment.
- **Route planner** — compare "Fastest" and "Most cycling friendly" routes between any two points, side-by-side, with distance, time, and elevation gain for each.

## Try it

Live at: **[YOUR-DEPLOYMENT-URL-HERE]**

Best viewed on desktop. The sidebar is 340 px wide; mobile is functional but cramped.

## Data sources

All data is loaded live from public APIs at page load. No data is bundled with the app.

| Layer | Source | Format |
|---|---|---|
| Bikeways | Brisbane City Council ArcGIS FeatureServer | GeoJSON |
| Cyclist crashes | QLD Government CKAN DataStore (resource `e88943c0…`) | JSON, points |
| Road hierarchy | BCC Opendatasoft (City Plan 2014 overlay) | GeoJSON |
| Parks | BCC Opendatasoft (Park Locations) | GeoJSON |
| Bicycle racks | BCC Opendatasoft (Bicycle Rack locations) | GeoJSON |
| LGA boundary | QLD Spatial Information (Admin Boundaries Framework) | GeoJSON |
| Live traffic | QLDTraffic API v2 (via corsproxy.io) | GeoJSON |
| Routing | OpenRouteService Directions v2 (cycling-regular) | GeoJSON |
| Map tiles | CARTO Dark Matter | Raster |

## Stack

- **MapLibre GL JS 4.7.1** for map rendering (loaded from jsDelivr CDN)
- **Vanilla JavaScript** — no framework, no build step, no npm dependencies
- **`sessionStorage`** for client-side caching of static datasets between page reloads
- **`fetch` with `AbortController`** for timed-out API requests
- **Single HTML file** (~80 KB minified) — deploy by uploading one file

## Limitations (read these)

These are real constraints of the data, not bugs:

- **The crash heatmap shows where crashes were recorded, not where risk is highest.** Roads with no recorded crashes may simply be roads no one cycles on. Treat heatmap density as the numerator of risk, not risk itself.
- **The road hierarchy reflects City Plan 2014.** Updates to the planning instrument are infrequent. Some "future" categories may have been built; some classifications may have changed in the decade since.
- **Crash data ends in mid-2024.** The QLD dataset is published with significant lag for verification. "Recent" cyclist crashes will not appear.
- **Routes come from OpenRouteService using OSM data; bikeways come from BCC.** The two sources will sometimes disagree on whether a given road carries cycling infrastructure. Both layers are shown so users can judge.
- **"Most cycling friendly" is ORS's recommended cycling preference — it biases toward OSM-tagged cycling infrastructure.** It is not a safety rating, not trained on crash data, and makes no claim about lowest stress.
- **Traffic events are filtered for "cyclist relevance" by heuristic, not by an authoritative field.** QLDTraffic does not tag events by cyclist impact. The score weights flooding, closures, and roadworks higher than motorway-only incidents. Verify locally before relying on it.
- **Bicycle rack coverage is sparse outside the CBD.** BCC's dataset reflects what the council has audited, not every rack in Brisbane.
- **VeloGrid is a research and visualisation tool, not a navigation app.** Do not use it to plan a real ride without verifying conditions locally.

## Deployment

Single-file deployment. To host your own copy:

1. Generate a free [OpenRouteService API key](https://openrouteservice.org/dev/#/signup) (2,000 requests/day, no credit card)
2. Open `index.html`, find `ORS_API_KEY = "..."`, paste your key
3. Restrict the key by HTTP Referer in the ORS dashboard to your deployment domain
4. Upload `index.html` to any static host (GitHub Pages, Netlify, Cloudflare Pages — all free for this scope)

The QLDTraffic public API key is included in the source. It's shared and rate-limited globally — fine for low-traffic deployments.

## Architecture notes

- **All data loaders run in parallel** via `Promise.allSettled` so one failure doesn't block others.
- **Caching is versioned** — bumping `CACHE_VERSION` invalidates all prior `sessionStorage` entries automatically.
- **MapLibre style expressions** colour features lazily — no per-feature property mutation.
- **Point-in-polygon containment** filters traffic events against the loaded LGA boundary, not a bounding box, so events in adjacent LGAs (Logan, Moreton Bay) are correctly excluded.
- **Routes are fetched in parallel** and rendered with distinct line colours; identical-length routes (common in suburbs with one practical path) are flagged with a note.

## Licence

The code is released under the MIT licence. Data is owned by its respective publishers — see each source link above for licensing.

## Acknowledgements

Built with public open data from Brisbane City Council, the Queensland Department of Transport and Main Roads, and OpenStreetMap contributors (via OpenRouteService and CARTO basemap tiles). Map rendering powered by MapLibre GL JS, an open-source fork of Mapbox GL JS.
