# Routeprint — architecture

A transparent travel-footprint calculator whose every number traces back to a cited source,
starting with flights and built to grow into rail, road and a map-based trip planner.

Status: draft, updated 2026-11-14 after the docs review. Owner: Menion. Background:
[flight-footprint-landscape.md](flight-footprint-landscape.md). UX: [design.md](design.md).

## Principles

| Principle | Consequence for the architecture |
|---|---|
| Every number is traceable | The engine returns a calculation trace (each step, value, unit, source id), not just a total. |
| Methodology is data, not code | Factors live in versioned, cited parameter files; code only applies them. |
| Assumptions are visible and switchable | Contested choices (non-CO2 multiplier, distance correction, allocation) are named parameters with alternative values, each cited. |
| Reproducible | A result URL pins the methodology and data versions. Every engine behaviour that changes an output belongs to the methodology version, so the same input and versions always give the same output. |
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
| Great-circle distance | haversine, mean Earth radius 6,371.0088 km | — | IUGG mean radius; formula pinned per methodology version |
| Distance correction | route/country factor, else 1.052 | great-circle (1.0) | Teoh et al., ADS-B flight tracks (Zenodo record 8369564) |
| Non-CO2 multiplier | **1.7** | 1.0 (CO2 only: excludes non-CO2), 3.0 (high) | 1.7: UK Government. 3.0: Lee et al. 2021, net aviation ERF ÷ CO2 ERF (100.9 / 34.3 ≈ 2.9). Teoh et al. 2024 argues for lower contrail values |
| Default aircraft per distance band | **proposed:** Airbus A320 (ceo) below 4,500 km flown; Boeing 787-9 from 4,500 km | candidates to test: B737-800, A320neo; A330-300, B777-300ER, A350-900 | Own choice, calibrated against Google TIM (A9); all candidates are direct EEA matches per the TIM README, Appendix A |
| Seats per aircraft and cabin | typical two-class layout per type | — | Manufacturer typical layouts, cited per type |
| Cabin class weights | seat-area ratio per cabin class: narrow-body 1 / 1 / 1.5 / 1.5, wide-body 1 / 1.5 / 4 / 5 (economy / premium / business / first) | — | IATA RP 1726, as documented in the Google TIM README |
| Load factor | 84.5 % global default; published regional averages where available | — | ICAO / IATA published averages; US: BTS T-100 (public domain); 84.5 % default as used by Google TIM (FRED) |
| Cargo share | cargo mass fraction per route type | — | BTS-derived, as Google TIM does (to verify) |

The non-CO2 default is provisional and will be revisited during the methodology work (see A6).

### 2. Calculation engine

- Pure library: `calculate(trip, methodologyVersion) -> Result`. No I/O, no network, no UI.
- `Result` = totals per scope layer + `trace[]`. Layers are reported separately; in v1 only the
  non-CO2 assumption is switchable (`nonco2` in the URL):
  1. Fuel combustion (TTW CO2)
  2. Fuel production (WTT)
  3. Non-CO2 effects (contrails, NOx, H2O)
  4. Life cycle: aircraft manufacture, airport infrastructure, in-flight services, luggage
     (phase 2)
- Trip model: `Trip { legs: Leg[], passengers, cabin }`, `Leg { mode, from, to, vehicle? }`. A
  stop is always two legs. Flight legs carry an optional aircraft type; missing values fall back
  to documented defaults, and the fallback itself appears in the trace. No airline input in v1:
  no open dataset makes the result depend on it.
- Flight fuel = LTO (take-off and landing, fixed per flight per aircraft type) + CCD (climb, cruise,
  descent, per distance band, interpolated), following EEA Tier 3. Both are separate trace steps,
  so the short-flight effect is visible in the explain view.
- Aircraft: an optional aircraft type per leg; without it, the documented default aircraft for the
  distance band. The choice is a trace step. Google TIM's typical figure for the route is shown
  next to the result as an independent comparison, never used as input (section 4).
- Engine behaviour that affects outputs (distance formula, interpolation, fallback order,
  rounding) belongs to the methodology version: a change that alters any output ships as a new
  `m`, older versions keep their own code path, and CI holds exact expected outputs per retained
  version.
- Ranges: the engine evaluates low / default / high parameter sets in one pass so the UI can show
  a range without a second call.
- Runs in the browser; the same package can run server-side later (API, batch validation).

### 3. Reference data

| Dataset | Use | Licence |
|---|---|---|
| OurAirports | Airport coordinates, IATA/ICAO codes, names | Public domain |
| EEA/EMEP Guidebook 2023, 1.A.3.a Aviation annex | Fuel burn per aircraft type and distance (LTO + CCD) | EEA reuse policy, attribution |
| Teoh et al. ADS-B distance statistics (Zenodo record 8369564, referenced by the Google TIM README) | Route/country distance correction; also the list of origin–destination pairs actually flown | CC BY 4.0 |
| Google TIM API emission figures | Typical per-class figure per route, as comparison and CI cross-check (A3) | CC BY-SA 4.0 — share-alike for files built from them |
| Aircraft type codes and families | Fallback rules (winglets, previous generation), "newer aircraft" what-if | TIM appendix / ICAO Doc 8643 |
| Seat layouts per aircraft type | Seats per cabin for the allocation step | Manufacturer publications, cited per type |
| Load factor and cargo share | Allocation step | ICAO / IATA published averages; BTS T-100 (US, public domain) |

Each data file carries its sources' licence in `REUSE.toml`: route files CC BY-SA 4.0 (TIM figures)
together with CC BY 4.0 (distance statistics), `aircraft.json` with EEA attribution,
`airports.json` / `coords.json` public domain.

#### Data delivery: a read-only database baked into the deploy

All reference data is read-only and changes rarely, so v1 needs no database server. A build-time
pipeline turns raw datasets into validated, versioned static files served from the CDN:

```
raw datasets ──> data-pipeline/ (TS scripts: filter, join, validate) ──> public/data/<data-version>/
(OurAirports CSV,                                                         coords.json
 EEA tables,                                                              airports.json
 distance statistics,                                                     aircraft.json
 TIM API snapshots)                                                       routes/<IATA>.json
```

| File | Content | Estimated size (gzip) | Loaded |
|---|---|---|---|
| `coords.json` | IATA code + lat/lon only | ~30–40 KB | Eagerly on the planner: the first result needs it, also from a shared link |
| `airports.json` | Airports with IATA code and scheduled service: codes, name, city, country | ~100–150 KB | Lazily, on first input focus (autocomplete) |
| `aircraft.json` | EEA fuel burn per type and distance band, seats, fallback rules | ~50 KB | With the first calculation |
| `routes/<IATA>.json` | Per origin airport: route distance factors, Google TIM per-class figures | a few KB each | On demand, one file per origin |

Sizes are estimates to confirm with the first pipeline run; the planner's data bytes are part of
the performance budget (design.md).

- **Distance is code, not data:** haversine with a fixed mean Earth radius, pinned in the
  methodology, runs client-side in microseconds. Only the correction factors are data.
- **Route list:** the pipeline queries Google TIM (up to 1,000 markets per request) for the
  origin–destination pairs in the distance statistics, i.e. routes actually flown. Markets without
  a TIM figure simply have no comparison row.
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
which is proprietary. Google's typical-emissions API does not close it: it returns grams per
passenger per cabin class only, no aircraft types, so it cannot drive the LTO + CCD trace.

| Need | Source |
|---|---|
| Result when no aircraft is chosen | Own engine with the documented default aircraft per distance band |
| Typical figure for the route, for comparison | Google TIM API per-class figure, snapshotted at build time |
| Explain, what-if, chosen aircraft, non-CO2, life cycle | Own engine on EEA data |
| "Newer aircraft" lever | What-if against the newer sibling of the assumed aircraft (ICAO Doc 8643 family), not a claim about the route |
| Validation | Own engine vs TIM snapshots on a fixed route sample, run in CI |
| Real per-flight data (future) | EU Flight Emissions Label, as airline participation grows |

Owning the engine is what makes the trace and switchable assumptions possible; TIM is the
independent check, shown to users and enforced in CI.

### 5. Web app

- Static site, client-side calculation, deployable to any CDN. No accounts in v1.
- State in the URL: every result is a shareable, reproducible link (see "Shareable links" below).
- Map: MapLibre GL with self-hosted OpenStreetMap vector tiles (PMTiles on Firebase Hosting, A8),
  so no third-party tile host sees visitors; "© OpenStreetMap contributors" always visible on the
  map. Flights drawn as great-circle arcs.
- The explain view is generated from the engine trace, so it can never drift from the calculation.

#### Shareable links

A shared link opens the page with identical inputs, identical assumptions and an identical
result, now and years later. The URL carries the inputs and the pinned versions; the result is
recomputed by the deterministic engine, never stored in the URL (shorter, and cannot be edited to
show a fake number).

```
https://routeprint.org/en/flight?legs=PRG-JFK,JFK-PRG&cabin=J&pax=2&m=2026.1&d=2026.4
```

Business class, two passengers, return; the default non-CO2 setting is omitted.

| Parameter | Meaning | Format / default |
|---|---|---|
| `legs` | Ordered legs, comma-separated; a stop is always two legs (`PRG-FRA,FRA-JFK`) | `FROM-TO` IATA codes, optional `:TYPE` ICAO aircraft type (`PRG-JFK:B789`) |
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
  figures where reproducible), plus exact expected outputs per retained methodology version.
- CI cross-check against TIM snapshots on a fixed route set; deviations above a threshold fail the
  build and must be explained in the methodology changelog. ICAO ICEC 2026 (CO2 only, has an API)
  is a candidate second baseline.
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
| A8 | Map tiles | Self-hosted PMTiles (OpenStreetMap extract) on Firebase Hosting; OSMF vector tile service; commercial provider | **Decided: self-hosted PMTiles** (2026-11-14). Keeps "no tracking" true (no third-party tile host), no dependency on a best-effort community service. Storage and extract size to check. |
| A9 | Result when no aircraft is chosen | Own default aircraft per distance band, Google TIM as comparison; Google TIM figure as headline with an illustrative breakdown | **Decided: own default per band** (2026-11-14). The trace then always sums to the headline; TIM stays an independent check. **Proposed reference types:** A320 (ceo) below 4,500 km, B787-9 above — mainstream current aircraft, not the newest, so the default does not flatter. **Calibration rule:** per band, keep the candidate whose results best match Google TIM's per-class figures over a fixed route sample (TIM reflects the fleet actually flying); split the band or move the 4,500 km boundary if no single type fits. A regional band (ATR 72, Dash 8-400, E195) for flights under ~600 km is added with the rail comparison (phase 3). |
