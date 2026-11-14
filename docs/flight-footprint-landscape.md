# Flight carbon footprint calculators — landscape review

Survey date: 2026-11-05. Owner: Menion. Project: Routeprint.

**Verdict:** no existing site combines trustworthy sourcing, exposed assumptions and decision-oriented
usability. The best data foundation (Google TIM) has a deliberately minimal UI and omits non-CO2
effects; the best UX (offset sellers) is a payment funnel; the most comprehensive published method
(ATP-DEC) has no public tool. There is a real gap.

## 1. What exists

### Offset sellers with a calculator attached

| Tool | Scope | Notes |
|---|---|---|
| [atmosfair](https://www.atmosfair.de/en/offset/flight/) | CO2 + altitude-weighted non-CO2 (×3 above 9 km) | Most granular consumer tool: aircraft type, cabin, scheduled vs charter, via-airports, airline comparison. Widely recommended by sustainable-travel experts. Calculator sits in an embedded frame; fields are fiddly. German NGO, since 2003. |
| [myclimate](https://co2.myclimate.org/en/flight_calculators/new) | CO2e with flat RFI 2.0 | Cleanest UI of the group. Rounds results to nearest 100 kg. Stepper is explicitly Calculate → Choose → Pay. |
| Sustainable Travel International | 140–550 g CO2e/pkm by class and haul | Best for private/chartered aircraft. Reports in tonnes, which flattens perception. |
| ecologi, Carbonmark, various | DEFRA/ICAO tiered factors | Thin wrappers around published factors; transparency varies. |

Common problem: the number is the top of a sales funnel, so a higher number is commercially
convenient. That alone costs them credibility with a sceptical user.

### Institutional / industry baselines

- **ICAO Carbon Emissions Calculator (ICEC)** — the UN reference, CO2 only, no non-CO2 effects.
  **Currently offline.** `applications.icao.int/icec` serves a maintenance notice saying it will
  resume "by the end of February 2026" — still down on 2026-11-05. The most-cited neutral baseline
  is simply not available right now.
- **IATA CO2 Connect** — the only one sourcing fuel burn and load factors directly from airlines,
  built on IATA RP 1726. Conflict of interest (airline trade body) and limited public documentation.
- **[EU Flight Emissions Label](https://www.flightemissions.eu/en)** (Regulation (EU) 2024/3170) —
  the best data provenance of anything here: real reported airline performance, actual fuel burn,
  actual passenger/cargo load, actual SAF purchases, EN ISO 14083. But it is voluntary and airline
  opt-in (Air France-KLM first mover, July 2025), and it is a label embedded in booking flows, not
  a public calculator. Worth tracking as a future data source, not as a competitor.

### Google Travel Impact Model — the strongest free option

[travelimpactmodel.org](https://travelimpactmodel.org/) · [github.com/google/travel-impact-model](https://github.com/google/travel-impact-model)

- EEA Tier 3 (Annex 1.A.3.a Aviation 2023) fuel burn, per aircraft type, LTO + CCD phases, with
  interpolation across distance bands and documented fallbacks for missing types.
- Distance adjustment from ADS-B flight tracking (Teoh et al., Imperial College): route-level
  factors, country-level fallback, +5.2% global default over great-circle.
- ISO 14083 lower heating value 43.1 MJ/kg, CORSIA carbon intensity → 3.1894 kg CO2e/kg TTW,
  0.6465 WTT, 3.8359 well-to-wake.
- CC BY-SA 4.0, public API, governance by an independent advisory committee (ICCT + Google).
- EASA Flight Emissions Labels override TIM estimates where they exist.
- **Omits contrails, NOx and short-lived climate pollutants from the headline CO2e.** Contrails
  appear only as coarse qualitative buckets inside Google Flights.
- Public calculator at `/lookup/route` is functional and deep-linkable
  (`?itinerary=PRG-JFK`) but deliberately bare: no explanation, no uncertainty, no guidance.

### Academic / NGO

- **TravelAndClimate.org** (Chalmers + Swedish tourism orgs) — the only decent multi-mode comparison
  (plane vs train vs car vs ferry, plus hotels). Flat 133 g CO2/pkm for air regardless of haul.
  Includes fuel production and non-CO2. Clunky; cabin class is hidden behind an aircraft icon.
- **IRAP travel-footprint-calculator** — built for research institutes, conference travel.
- **ATP-DEC** — [Nature Comms Earth & Environment, 29 Oct 2025](https://www.nature.com/articles/s43247-025-02847-4).
  The most comprehensive published method: NOx, H2O, contrail-induced cloudiness, carry-on and
  checked luggage, in-flight services, aircraft and airport life cycles, plus "historical adjustment
  factors" from a rolling window of real flight tracks (captures e.g. Russian airspace closure
  adding distance to LHR–PVG). Validated on 30,000+ flights. **No public web tool found** — it is a
  paper, not a product.

## 2. The actual problem is divergence, not usability

Same flight, different tools, roughly 2.5× spread. JFK–MXP round trip, economy:

| Tool | Result |
|---|---|
| ICAO | 716 kg CO2 |
| TravelAndClimate.org | 1,722 kg CO2e |
| Sustainable Travel International | 1,790 kg CO2e |

Measured directly during this review — PRG→JFK one way, Google TIM (well-to-wake CO2e per passenger):

| Economy | Premium | Business | First |
|---|---|---|---|
| 417.5 kg | 579.6 kg | 1,434.2 kg | 1,763.9 kg |

Note the class factor: business is 3.4× economy on the same aircraft. That is the single biggest
lever a traveller controls, and almost no calculator leads with it.

### Why the numbers diverge

1. **Scope.** CO2 only → CO2e (Kyoto gases) → + non-CO2 (contrails, NOx, H2O) → + life cycle
   (aircraft manufacture, airport operation, catering, luggage).
2. **Non-CO2 multiplier.** This is the dominant source of spread, and it is a *choice*, not a fact:
   - ICAO, IATA, Google TIM: 1.0 (excluded)
   - UK Government 2026 methodology: **1.7 central estimate**
   - myclimate: flat RFI 2.0
   - atmosfair: ×3 for emissions above 9 km
   - Lee et al. 2021: GWP100 ≈ 1.7, GWP20 ≈ 4.0 for total non-CO2
   - Teoh et al. 2024 (ACP): contrail cirrus RF ~44% lower than Lee et al.; GWP20 1.12 vs 2.32,
     GWP100 0.30 vs 0.63
   - Lee et al. themselves warn RFI is **not** an emissions multiplier — it is routinely misused
     as one.
3. **Distance.** Great-circle vs +5% ADS-B correction vs historical actual tracks.
4. **Allocation.** Cabin class weighting, load factor assumptions, passenger/cargo split.

## 3. Where a new site could win

Nobody currently does these:

1. **Show the range, not a single number.** Expose the non-CO2 assumption as a visible toggle
   (CO2 only / 1.7 / 3.0) with the source behind each. Honest uncertainty beats false precision,
   and it is the only defensible position given Lee vs Teoh.
2. **Decision-oriented, not accounting-oriented.** Rank what actually changes the answer: cabin
   class, aircraft generation, direct vs connecting, airline load factor. atmosfair half-does this,
   behind an offset funnel.
3. **No offsets, no donations, no sales.** The credibility differentiator.
4. **Every factor cited inline**, linked to the primary source, versioned.
5. **Mode comparison** (plane vs train vs car) with the same rigour — currently only
   TravelAndClimate.org, which is clunky and Sweden-centric.

### Suggested foundation

- **Base fuel burn and distance:** Google TIM. Open methodology, CC BY-SA 4.0, free API,
  aircraft-specific, ICCT-governed. Do not re-derive EEA Tier 3 fuel burn by hand.
  Licence: TIM repository is CC BY 4.0; TIM API figures are CC BY-SA 4.0 (share-alike).
- **Non-CO2 layer:** own, parameterised, citing Lee et al. 2021 and Teoh et al. 2024, defaulting to
  the UK Government 1.7 central estimate.
- **Scope extras** (luggage, in-flight services, airport/aircraft LCA): follow ATP-DEC's published
  method if you want the full picture; it is the best-documented comprehensive scope available.
- **Watch:** EU Flight Emissions Label as participation grows — real airline data beats any model.

## Decisions (2026-11-05)

| Topic | Decision | Status |
|---|---|---|
| Project name | **Routeprint** — mode-agnostic, map-oriented. "Wayprint" rejected: a live travel product already uses it (wayprnt.com). | Final |
| Non-CO2 default | **1.7** (UK Government central estimate), shown as a visible, sourced toggle next to CO2-only and a high-end value. | Provisional — revisit with the methodology work |
| Scope of v1 | **Flights only**, full well-to-wake plus non-CO2. Aircraft and airport life cycle follow in a later phase. | Final |
| Rail / transit comparison | **Later phase.** Needs a public-transit router from the start of that phase, so it is kept out of v1 and the architecture leaves room for it. | Final |
| Language | **English first**, every user-facing string and methodology text prepared for translation from day one. | Final |

See [architecture.md](architecture.md) and [design.md](design.md).
