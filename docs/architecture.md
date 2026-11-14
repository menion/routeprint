# Routeprint — architecture

A transparent travel-footprint calculator whose every number traces back to a cited source,
starting with flights and built to grow into rail, road and a map-based trip planner.

Status: draft for discussion, 2026-11-05. Owner: Menion. Background:
[flight-footprint-landscape.md](flight-footprint-landscape.md). UX: [design.md](design.md).

## Principles

| Principle | Consequence for the architecture |
|---|---|
| Every number is traceable | The engine returns a calculation trace (each step, value, unit, source id), not just a total. |
| Methodology is data, not code | Factors live in versioned, cited parameter files; code only applies them. |
| Assumptions are visible and switchable | Contested choices (non-CO2 multiplier, distance correction, allocation) are named parameters with alternative values, each cited. |
| Reproducible | A result URL pins the methodology version; same input and version always give the same output. |
| No commercial bias | No offsets, no affiliate links, no tracking that sells data. Funded by individual support only (GitHub Sponsors); no company sponsorship. |
| Private and cheap to run | Calculation runs client-side; v1 needs no backend and no accounts. |
| Mode-agnostic core | A trip is a list of legs; a leg has a mode. Flight is the first mode, not the only one. |

## System overview

```
                  +--------------------------------------------+
                  |                  Web app                   |
                  |  trip input · map · result · explain view  |
                  +----------------------+---------------------+
                                         |
                                         v
+----------------+   loads   +-----------------------+   uses   +------------------------+
| Methodology    |---------->|   Calculation engine  |<---------| Reference data         |
| (versioned,    |           |   pure, deterministic |          | airports, aircraft,    |
|  cited params) |           |   result + trace      |          | fuel-burn tables,      |
+----------------+           +-----------+-----------+          | distance factors       |
                                         ^                      +------------------------+
                         later phases    |  distance + geometry per leg
                     +-------------------+-------------------+
                     |                                       |
            +-----------------+                     +-----------------+
            | Transit router  |                     | Road router     |
            | (rail, bus)     |                     | (car)           |
            +-----------------+                     +-----------------+
```

## Components

### 1. Methodology (versioned parameters + bibliography)

- `methodology/<version>/` holds parameter files (YAML), e.g. `flight.yaml`.
- Every parameter has: id, value, unit, source id, optional page/table reference, optional
  alternatives (`{ value, label, source }`), and a one-line rationale.
- `sources/` holds the bibliography (CSL-JSON): one entry per paper, dataset or regulation.
- Versioned (`2026.1`, `2026.2`). Any value change is a new version with a changelog entry. Old
  versions stay loadable so shared links remain reproducible.

Initial flight parameters (v1):

| Parameter | Default | Alternatives | Source |
|---|---|---|---|
| Fuel → CO2 (TTW) | 3.1894 kg/kg | — | ISO 14083 LHV × CORSIA carbon intensity (as used by TIM) |
| Fuel → CO2e (WTT) | 0.6465 kg/kg | — | CORSIA LCA (as used by TIM) |
| Distance correction | route/country factor, else 1.052 | great-circle (1.0) | Teoh et al., ADS-B data (via TIM) |
| Non-CO2 multiplier | **1.7** | 1.0 (CO2 only), 3.0 (high) | UK Government; Lee et al. 2021; Teoh et al. 2024 |
| Cabin class weights | per aircraft seat map where known, else defaults | — | TIM / IATA RP 1726 (to verify) |
| Load factor, cargo share | regional averages | — | ICAO / EEA (to verify) |

The non-CO2 default is provisional and will be revisited during the methodology work (see A6).

### 2. Calculation engine

- Pure library: `calculate(trip, methodologyVersion) -> Result`. No I/O, no network, no UI.
- `Result` = totals per scope layer + `trace[]`. Scope layers, each switchable:
  1. Fuel combustion (TTW CO2)
  2. Fuel production (WTT)
  3. Non-CO2 effects (contrails, NOx, H2O)
  4. Life cycle: aircraft manufacture, airport infrastructure, in-flight services, luggage
     (phase 2)
- Trip model: `Trip { legs: Leg[], passengers, cabin }`, `Leg { mode, from, to, via?, vehicle? }`.
  Flight legs carry optional aircraft type and airline; missing values fall back to documented
  defaults, and the fallback itself appears in the trace.
- Flight fuel = LTO (take-off and landing, fixed per flight per aircraft type) + CCD (climb, cruise,
  descent, per distance band, interpolated), following EEA Tier 3. Both are separate trace steps,
  so the short-flight effect is visible in the explain view.
- Aircraft: an optional aircraft type per leg; without it, the typical fleet mix for the route
  (TIM snapshot), else a documented default per distance band. The choice is a trace step.
- Ranges: the engine evaluates low / default / high parameter sets in one pass so the UI can show
  a range without a second call.
- Runs in the browser; the same package can run server-side later (API, batch validation).

### 3. Reference data

| Dataset | Use | Licence |
|---|---|---|
| OurAirports | Airport coordinates, IATA/ICAO codes, names | Public domain |
| EEA/EMEP Guidebook 2023, 1.A.3.a Aviation annex | Fuel burn per aircraft type and distance (LTO + CCD) | EEA reuse policy, attribution |
| Google TIM distance factors | Route/country distance correction | CC BY 4.0 (TIM repository; verify per file) |
| Google TIM API emission figures | Typical-route snapshots (A3) | CC BY-SA 4.0 — share-alike for files built from them |
| Aircraft type codes and families | Fallback rules (winglets, previous generation) | TIM appendix / ICAO Doc 8643 |

#### Data delivery: a read-only database baked into the deploy

All reference data is read-only and changes rarely, so v1 needs no database server. A build-time
pipeline turns raw datasets into validated, versioned static files served from the CDN:

```
raw datasets ──> data-pipeline/ (TS scripts: filter, join, validate) ──> public/data/<data-version>/
(OurAirports CSV,                                                         airports.json
 EEA tables,                                                              aircraft.json
 TIM factors)                                                             routes/<IATA>.json
```

| File | Content | Estimated size (gzip) | Loaded |
|---|---|---|---|
| `airports.json` | Airports with IATA code and scheduled service: codes, name, city, country, lat/lon | ~100–150 KB | Lazily, on first input focus (autocomplete) |
| `aircraft.json` | EEA fuel burn per type and distance band, fallback rules | ~50 KB | With the first calculation |
| `routes/<IATA>.json` | Per origin airport: route distance factors, TIM typical-emission snapshots | a few KB each | On demand, one file per origin |

Sizes are estimates to confirm with the first pipeline run.

- **Distance is code, not data:** great-circle distance on WGS84 (haversine, or Vincenty for
  extra precision) runs client-side in microseconds. Only the correction factors are data.
- **Sharding** by origin airport keeps the file count in the low thousands, comfortably within
  static-host limits.
- **Versioning:** each result pins a data version as well as a methodology version, so old links
  stay reproducible. The pipeline records source URL, retrieval date and checksum per dataset.
- **Updates** mean rebuild and redeploy, e.g. quarterly or when a source publishes a new edition.

A server or database becomes necessary only for: live per-flight lookups (TIM API by date and
flight number), user accounts or saved trips, the public computation API, or transit routing
(phase 3, an external router service). Even then, the static files stay the source of truth for
reference data.

### 4. Flight data strategy

The hard gap: **which aircraft actually flies a route** comes from schedule data (OAG, Cirium),
which is proprietary. Proposed hybrid:

| Need | Source |
|---|---|
| "Typical" emissions for a route (scheduled fleet mix) | Google TIM API, snapshotted at build time |
| Explain, what-if, chosen aircraft, non-CO2, life cycle | Own engine on EEA data |
| Validation | Own engine vs TIM API on a fixed route sample, run in CI |
| Real per-flight data (future) | EU Flight Emissions Label, as airline participation grows |

Owning the engine is what makes the trace and switchable assumptions possible; TIM supplies what
open data cannot (fleet mix) and acts as an independent check.

### 5. Web app

- Static site, client-side calculation, deployable to any CDN. No accounts in v1.
- State in the URL: every result is a shareable, reproducible link (see "Shareable links" below).
- Map: MapLibre GL with OSM-based vector tiles; flights drawn as great-circle arcs.
- The explain view is generated from the engine trace, so it can never drift from the calculation.

#### Shareable links

A shared link opens the page with identical inputs, identical assumptions and an identical
result, now and years later. The URL carries the inputs and the pinned versions; the result is
recomputed by the deterministic engine, never stored in the URL (shorter, and cannot be edited to
show a fake number).

```
https://routeprint.org/en/flight?legs=PRG-JFK,JFK-PRG&cabin=Y&pax=1&nonco2=1.7&m=2026.1&d=2026.4
```

| Parameter | Meaning | Format / default |
|---|---|---|
| `legs` | Ordered legs, comma-separated; a connection is two legs | `FROM-TO` IATA codes, optional `:TYPE` ICAO aircraft type (`PRG-JFK:B789`) |
| `cabin` | Cabin class | `Y` economy (default), `W` premium economy, `J` business, `F` first |
| `pax` | Passengers | Integer, default 1 |
| `nonco2` | Non-CO2 assumption | One of the methodology's listed values; omitted = methodology default |
| `m` | Methodology version | Always written into shared links |
| `d` | Reference data version | Always written into shared links |

- **Pinning:** a hand-typed URL without `m`/`d` uses the latest versions; the page then rewrites
  its own URL (`history.replaceState`) so that copying it pins them.
- **Canonical form:** uppercase codes, fixed parameter order, defaults omitted except `m` and `d`
  — the same trip always produces the same link.
- **Newer version available:** the shared result is shown as calculated, with a notice offering
  the result under the current methodology and the difference.
- **Retention:** every methodology version and every data version stays deployed, otherwise old
  links would break. Methodology files are tiny; data versions are a few MB and published a few
  times a year. Revisit if that grows.
- **Robustness:** unknown parameters are ignored; invalid values (unknown airport) keep the rest
  of the input prefilled and show an inline error.
- **Locale** is in the path only; the same link in another language gives identical numbers.
- **Privacy:** the URL holds trip inputs only, no personal data and no tracking parameters.
- **Social previews:** v1 uses a generic preview image and title. Per-result previews (route map
  + number) need server rendering and come with the backend (section 8).
- **Short links** (`/s/<id>`) are not needed for v1, as links stay short; possible later with
  Firestore.

### 6. Internationalisation (from day one)

- No user-facing string in code: all UI text in message catalogs (ICU MessageFormat for plurals
  and units).
- Locale-aware formatting through `Intl` (numbers, units, dates). Calculation is always metric;
  imperial is a display option only.
- Methodology explanations and source summaries are content files per locale
  (`content/<locale>/...`), English as the reference; untranslated pages fall back to English.
- Layout tolerant of longer strings and RTL (logical CSS properties).
- Locale in the URL path (`/en/`, `/cs/`) for shareable, indexable pages.

### 7. Later phases — room left in the design

| Phase | Adds | Architectural hook |
|---|---|---|
| 2 | Aircraft and airport life cycle, luggage, in-flight services (ATP-DEC style) | New scope layer + parameters; no engine restructuring |
| 3 | Rail and bus comparison | `mode: rail`; transit router behind a `Router` interface. Candidates: Transitous (community MOTIS deployment, Europe-wide) or self-hosted MOTIS / OpenTripPlanner on GTFS. Rail factors per country electricity mix. |
| 3 | Car | `mode: road`; road router (Valhalla, GraphHopper or OSRM). Vehicle type and occupancy as parameters. |
| 4 | Trip planner on the map | Multi-modal itinerary comparison using the same engine and trace. |
| Later | Public computation API (e.g. for Locus Map) | Same engine behind an HTTP endpoint (serverless/edge function); request = `Trip`, response = `Result` with trace. |
| 5 | Any routing system (GraphHopper, Mapbox, Valhalla, MOTIS, …) | One `Router` adapter per provider; provider chosen per mode and region. Check each provider's terms on caching and displaying results on third-party maps. |
| 5 | Live flight and train offers | `OfferSource` adapters (flight search APIs, rail operator APIs) return candidate itineraries with real schedules, aircraft types and prices. Server-side only (API keys, caching, rate limits). |
| 6 | Carbon-optimised trip planner | Planner layer: routers + offer sources → candidate itineraries → engine → ranking. Shows the trade-off set (CO2e vs. time vs. price), not a single "greenest" answer. |
| 6 | Personal footprint tracking | Saved trips (section 8) become a log of trips actually taken, with totals per year. |

Routers and offer sources only supply itineraries, geometry and distance per leg; emissions always
come from the engine, so every mode and every provider is explained the same way.

```
 Router adapters ─────┐
 (GraphHopper, Mapbox, │      candidate       +--------+   Result + trace   +----------+
  MOTIS, Valhalla)     ├────> itineraries ───>| Engine |──────────────────> | Ranking  |
 Offer sources ────────┘                      +--------+   per itinerary    | CO2e /   |
 (flight, rail APIs)                                                         | time /   |
                                                                             | price    |
                                                                             +----------+
```

### 8. Growth path: accounts, saved calculations, blog

Static-first does not lock the site in. Astro renders pages statically by default and lets single
routes opt into server rendering (`export const prerender = false`) once a server adapter is
added; API endpoints live next to pages (`src/pages/api/*.ts`).

| Feature | Approach | Needs a server? |
|---|---|---|
| Blog | Markdown content collection in the repo, built statically, translatable like methodology pages | No |
| Login | Firebase Authentication (email link, Google, Apple); ID token verified server-side | Only for token verification |
| Saved calculations | Firestore, one document per saved trip: input `Trip` + methodology and data version, **not** the computed result; recomputed on view so it stays reproducible | Yes — API endpoints |
| Public computation API | Same TS engine in an endpoint | Yes |

Hosting on Google Cloud:

- **Firebase Hosting** serves the static pages and data files through its CDN.
- **Cloud Run** runs the Astro Node adapter for the few server routes and the API; Firebase
  Hosting rewrites `/api/**` and the account pages to it. Scales to zero, so it costs nothing
  while unused.
- Firestore and Firebase Auth are pay-per-use with free tiers; no always-on database instance.

Rule for v1 so the growth path stays cheap: keep the engine pure, keep reference data in static
files, and use no host-specific APIs in the app code.

## Quality and verification

- Golden tests against published worked examples (TIM fuel table, EASA FEL examples, ATP-DEC
  figures where reproducible).
- CI cross-check against TIM snapshots on a fixed route set; deviations above a threshold fail the
  build and must be explained in the methodology changelog.
- Every parameter file validated against a schema: no value without a source id.

## Open decisions

| # | Question | Options | Proposal |
|---|---|---|---|
| A1 | Engine language | TypeScript; Kotlin Multiplatform compiled to JS | **Decided: TypeScript.** Website only; KMP's benefit is native reuse, which is not planned. A future computation API (e.g. for Locus Map) runs the same TS engine server-side. |
| A2 | Web framework | Vite + light UI library; SvelteKit (static); Astro with interactive islands | **Decided: Astro.** Content-heavy (methodology, blog, i18n) with islands for calculator and map; per-route server rendering covers the growth path (section 8). |
| A3 | TIM API usage | Build-time snapshots; live via cached proxy | Build-time snapshots for v1: no backend, no API key in the client |
| A4 | Licence | Code: MIT / Apache-2.0 / AGPL / MPL | **Decided.** Public GitHub repository. Code **Apache-2.0**; own text (methodology, explainers, blog) **CC BY 4.0**; data files built from TIM API figures **CC BY-SA 4.0**. Per-path licences following the REUSE convention (`LICENSES/` + SPDX tags). |
| A5 | Hosting | Firebase Hosting (+ Cloud Run later); Cloudflare Pages; Netlify | **Decided: Firebase Hosting.** Static today; the Google Cloud backend (section 8) attaches without migrating hosts. |
| A6 | Non-CO2 application | Multiplier on TTW CO2; on WTW; per-effect model (ATP-DEC) | Multiplier on TTW for v1; per-effect model as a later methodology version |
| A7 | Independence vs. offer integrations (phase 5+) | Plain links out to booking sites; affiliate partnerships; no booking links at all | Plain links out by default. If a flight or rail API is only available under affiliate terms: disclose it on the page, and keep ranking rules public and commission-blind. Decide before phase 5. |
