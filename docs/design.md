# Routeprint — product and UX design

What a person sees and does on Routeprint: get an honest footprint for a trip in seconds, then
understand where every kilogram comes from and what would change it.

Status: draft, updated 2026-11-14 (D1, D4, D6 decided; docs review applied). Owner: Menion.
Architecture:
[architecture.md](architecture.md).

## Audience

| Who | Wants | Serves them | Priority |
|---|---|---|---|
| Everyday traveller | "How bad is this trip, and what can I do about it?" — in plain words, in seconds | Landing input, headline result, levers | **Primary** |
| Curious or sceptical reader | "Why should I trust this number?" | "?" icons and "Read more" on the result, explain view | Secondary |
| Journalist, teacher, researcher | Citable, reproducible figures | Shareable links, methodology pages, GitHub source | Secondary |
| Organisation (later) | Travel-policy numbers | Batch / API (not v1) | Later |

The site is designed for common users, not for labs or scientists. Depth is never in the way:
methodology and calculation details are one tap away for those who want them, and complete in the
public GitHub repository.

### Layers of detail (progressive disclosure)

| Layer | Where | Content |
|---|---|---|
| 1. Answer | Result card | One number, plain-language range, one sentence of meaning |
| 2. Hint | "?" icon next to a number or control | Two or three sentences: what it is, why it matters |
| 3. Read more | Expandable section or explain view | Step-by-step derivation with values and short source names |
| 4. Full detail | Methodology and Sources pages, GitHub repository | Parameter files, sources, changelog, engine code |

Words on layers 1–2 avoid jargon: "climate effects of contrails and other high-altitude
emissions", not "non-CO2 ERF multiplier". Technical terms appear from layer 3 on.

## Design principles

1. **Answer first, depth on demand.** One number in seconds; the full derivation one click away.
2. **A range, not false precision.** Show the default with its low–high range; round to two
   significant figures (`420 kg`, not `417.5 kg`). Exact values live in the explain view.
3. **Every number is referenced and verified** — headline, range, breakdown, lever deltas and
   comparisons alike. Each has a "?" icon with its derivation and source, never forced on the
   user. Where an independent source exists, the popup also shows the cross-check (e.g. "Google
   TIM: 418 kg — matches").
4. **Assumptions are on the surface.** The non-CO2 choice is a visible control in plain words,
   not a footnote.
5. **Neutral voice.** No guilt, no green-washing, no offset upsell. State facts and comparisons.
6. **Works for everyone.** Mobile first, keyboard and screen-reader accessible (WCAG 2.2 AA),
   readable in light and dark.
7. **Modern, but fast.** Contemporary look without heavy frameworks or large images. A few small
   animations are welcome (result number counting up, breakdown bar filling, map arc drawing,
   switch transitions) — optional, sparse, and off under `prefers-reduced-motion`. Speed is a
   feature (see "Performance budget").
8. **Shareable by default.** The URL is the state; any result can be sent and reproduced.

### Performance budget

| Metric | Target |
|---|---|
| Largest Contentful Paint, mid-range phone on 4G | < 2.0 s |
| JavaScript on the landing page | < 50 KB gzip |
| JavaScript on the planner page, excluding the map | < 120 KB gzip |
| Data on the planner's critical path (coordinates, route file, aircraft table) | < 100 KB gzip |
| Map library and tiles | Loaded lazily, after the result is visible |
| Fonts | System font stack or one subsetted variable font |
| Lighthouse performance and accessibility | ≥ 95 |

## Information architecture

The site is multipage by nature (Astro), and will grow: methodology, learn, blog, later accounts.
The calculator is split over two pages (D6): a light landing page and a separate planner page.

| Page | Purpose |
|---|---|
| Landing | Short intro, the From → To input and three short value points; CTA to the planner. No map, almost no JavaScript |
| Planner / result | Result, assumption switch, map, levers — the page shareable links point to |
| Explain (per result) | Step-by-step derivation generated from the engine trace |
| Methodology | Human-readable description of each methodology version, with changelog |
| Sources | Bibliography: every paper, dataset and regulation used, with what it is used for and its licence (incl. the CC BY-SA notice for Google TIM figures) |
| Learn | Short explainers: non-CO2 effects, why calculators disagree, class and aircraft impact, why short flights use more fuel per km |
| About | Who, why, funding, independence statement, licence |
| Privacy | What "No tracking" means (D7), what the hosting provider logs, data attributions |

The "How it works" item in the main navigation opens the Methodology page.

## Core flow (v1, flights)

Landing page:

```
How much does your flight really warm the climate?
[From ...........] [To ...........]   [ See the impact ]
+ Add stop    More precise: choose aircraft
-------------------------------------------------------------- (smaller second row)
Trip [One way | Return]   Cabin [Economy ▾]   Passengers [1 ▾]
```

Planner page (desktop: two columns; mobile order in brackets):

```
+---------------------------------------------------------------+
| Prague (PRG) → New York (JFK)                     [Edit trip] |  summary bar
| One way · Economy · 1 passenger                               |
+-------------------------------------+-------------------------+
| (1) RESULT                          | (3) MAP                 |
| 660 kg CO2e  per passenger, one way |                         |
| [plain sentence giving meaning, D3] | (2) WHAT CHANGES THIS   |
| Range 0.42 – 1.1 t …            (?) | Business   2.3 t +240%  |
| [CO2 only | Recommended | High]     | One stop   710 kg +8%   |
| ████████ ███ ██████  breakdown      | Newer type 560 kg −15%  |
| [How is this calculated?] [Share]   |                         |
| (4) CALCULATION (collapsed)         | (5) IN CONTEXT          |
+-------------------------------------+-------------------------+
```

Notes (numbers illustrative: TIM economy PRG→JFK 417.5 kg WTW, non-CO2 1.7 on TTW):

- Landing: From, To and the CTA in the main row; trip type, cabin and passengers in a smaller
  second row with defaults (one way, economy, 1), so the start is not overwhelming but nothing is
  hidden.
- Planner: the trip collapses into a one-line summary bar with "Edit trip"; the result is the
  first thing visible, also on mobile.
- Mobile order on the planner: result → what changes this → map → calculation → in context.
- "+ Add stop" adds a Stop field between From and To; the stop shows on the map and in the
  calculation, and "Fly direct" appears as a lever.
- Airport input accepts city names, airport names and IATA codes, with fuzzy matching and
  keyboard navigation. Typing "Prague" must just work.
- The optional aircraft type sits under "More precise" and is never required. No airline field in
  v1: no open data makes the result depend on it.
- Defaults are announced, not hidden: "Aircraft: typical for this distance (Boeing 787-9)",
  following the default aircraft per distance band (architecture A9).
- Footer on every page carries the trust line "Methodology 2026.1 · data version 2026.4 · No
  offsets · No tracking" and links to the Privacy page, so the versions behind every number and
  the meaning of "No tracking" are always one click away.

## Result anatomy

| Element | Rule |
|---|---|
| Headline number | Default methodology, two significant figures, unit and per-passenger scope always stated |
| Meaning line | One plain sentence comparing the number with a sourced reference point (needs D3) |
| Range | Low–high from the switchable assumptions, shown next to the headline, never hidden |
| Assumption switch | Three short labels — "CO2 only · Recommended · High" — the full explanation behind each "?" |
| Breakdown bar | Stacked by scope layer (fuel burn, fuel production, non-CO2). Until phase 2, the legend lists "Aircraft and airport life cycle — not counted yet" with a "?", and no bar segment, so the full scope is visible without implying a number |
| Precision note | What was assumed because the user did not specify it |

## Explain view

Collapsed by default; opens from "How is this calculated?" or any "Read more". Generated from the
engine trace; one row per step, in calculation order:

| Step | Value | Source |
|---|---|---|
| Great-circle distance PRG → JFK | 6,550 km | Haversine on OurAirports coordinates |
| Route distance correction | × 1.05 | Teoh et al., ADS-B flight tracks, route factor |
| Fuel: take-off and landing (LTO, fixed per flight) | … kg | EEA Guidebook 2023, LTO table |
| Fuel: climb, cruise and descent (CCD, by distance) | … kg | EEA Guidebook 2023, CCD table, interpolated |
| Fuel per passenger (seats, load factor, cabin weight) | … kg | Methodology parameters, each cited |
| … | … | … |
| Comparison: Google TIM typical for this route | … kg | Google TIM 2026 (CC BY-SA 4.0); independent, not an input |

- Fuel is always shown as its two phases, so the short-flight effect is visible: the fixed
  take-off and landing fuel is spread over fewer kilometres, and the climb is a larger share of a
  short trip. A short panel under the steps shows fuel per km at three distances for one aircraft
  (e.g. Boeing 787-9: 8.1 kg/km at 926 km, 5.9 kg/km at 9,260 km).
- Each source links to the Sources page entry and, where possible, the exact table or page.
- A "Methodology version 2026.1" badge links to the methodology page and changelog.
- Copy-as-text and CSV export (low priority; researchers also have the GitHub source).

## Sharing

Every result can be shared as a link that reopens the page with the same inputs, assumptions and
result. Link format and versioning: [architecture.md](architecture.md), "Shareable links".

- **Share button** next to the result: native share sheet on mobile (Web Share API), copy to
  clipboard with a short confirmation on desktop. The address bar always holds the same link, so
  copying it directly works too.
- **Opening a shared link** shows the filled-in form, the result with the same assumption switch
  position, and scrolls to the result. Nothing to click to see the number.
- **Deep link to the explain view:** `#explain` opens the derivation directly, for citing in
  articles and discussions.
- **Older methodology:** a calm notice, not a warning: "Calculated with methodology 2026.1. With
  the current version 2026.3: 640 kg (−3%). Show current."
- **Previews** in chat apps and social media: generic Routeprint card in v1; a per-result card
  (route + number) once server rendering exists.

## Levers ("What changes this")

Ranked by size of effect for this specific trip, not a fixed list. Candidate levers in v1:

- Cabin class
- Direct vs. connecting (with the extra distance and the extra take-off/landing cycle)
- Newer aircraft: what-if against the newer sibling of the assumed aircraft (e.g. A320ceo →
  A320neo), not a claim about which aircraft fly the route
- Return vs. one-way framing

Each lever shows the resulting value and the change in percent ("Business class: 2.3 t, +240%"),
with a "?" popup proving it: the two values
compared, their source, and why they differ (e.g. business +240%: Google TIM per-class figures for
this route, 1,434 kg vs. 417.5 kg, because a business seat takes several times the cabin area of an
economy seat). "Read more" opens the full explain view for the alternative.
Rail alternatives appear here in phase 3.

## Context and comparisons

Comparisons are easy to get wrong and easy to use for manipulation, so:

- Only sourced reference points (e.g. per-capita annual emissions by country, a per-capita budget
  consistent with a stated temperature target), each with its source and year.
- No vague equivalents ("= 3 trees") unless they come from a cited, defensible calculation.
- Reference points are methodology parameters too, so they are versioned and translatable.

## Map

- MapLibre GL with self-hosted OpenStreetMap tiles (architecture A8), calm and low-contrast base
  so the route is the focus. "© OpenStreetMap contributors" is always visible on the map, never
  hidden behind a toggle.
- Flights as great-circle arcs; connections as separate arcs with stop markers.
- Optional "actual flown distance" note rather than a fake track.
- Map is supporting, not required: the result is fully usable without it (and for screen readers).

## Visual tone

- Calm, factual, editorial. Closer to a good data-journalism piece than to a campaign site.
- No red-alarm colours for results; use one blue scale for magnitude and reserve other colours for
  categories in the breakdown.
- Numbers in tabular figures; conservative, compact spacing — information-dense but uncluttered;
  one typeface family with a clear numeric style.
- Charts colour-blind safe and readable in both themes.

### Colour direction

- **Blue-led, pastel, not too bright.** Soft, slightly desaturated blues (powder, mist, steel) for
  surfaces and fills; a deeper muted blue for primary actions and links.
- **Contrast first:** pastels are for backgrounds, cards and chart fills only. Text and interactive
  elements use darker tones that meet WCAG AA (4.5:1 for body text).
- **Breakdown categories:** distinguishable within the calm palette — e.g. two blues plus a muted
  sand or sage — and checked for colour blindness.
- **Dark theme:** deep desaturated navy background, the same pastels slightly dimmed.
- Exact tokens come out of the prototype (D5).

## Internationalisation (UX side)

- English first; locale switcher prepared but hidden until a second locale is complete.
- Units: metric by default, imperial as a display preference; kg/t switch at a consistent
  threshold in every locale.
- Avoid text in images; allow 30–40% string expansion; test with a pseudo-locale in CI.

## Support and funding

Routeprint is funded by **individual support only, through GitHub Sponsors**. No company
sponsorship, no advertising — one airline or offset-seller sponsor would undermine trust in every
number on the site.

| Placement | Content |
|---|---|
| Footer, every page | "Support Routeprint" link, same weight as other footer links |
| About → "How Routeprint is funded" | Running costs, income, the individual-only policy, link to GitHub Sponsors |
| End of the explain view | One line: "This calculation is free and open. If it helped you, you can support the work." |

- **Never** next to the result number or the share button, never in a popup or modal, no reminders
  after N visits. Next to a footprint figure, any money button reads as "compensate your flight".
- Wording: "Support Routeprint", not "Donate". The support page states: "This supports the
  website. It does not offset your flight."
- Ways to help without money sit alongside: star on GitHub, share a result, report a data error.

## Out of scope for v1

- Accounts, saved trips, trip history.
- Offsets of any kind.
- Rail, bus, car (phase 3); map trip planner (phase 4).
- Life-cycle layer (phase 2).

## Open decisions

| # | Question | Proposal |
|---|---|---|
| D1 | Default display: one-way or return? | **Decided: one way** as the default; return is one tap away and doubles the trip |
| D2 | Headline unit for long trips: kg or t? | kg below 1,000, t above, consistently. Round to two significant figures first, then pick the unit (995 kg → "1 t"). A range uses the unit of its upper end ("420 kg – 1.1 t" becomes "0.42 – 1.1 t") |
| D3 | Which context reference points? | **Priority:** they also feed the meaning line under the headline, the part common users need most. Decide with the methodology work; each must be sourced |
| D4 | High-estimate label for non-CO2 3.0 | **Decided: "High"** as the short switch label; the full wording sits behind its "?" |
| D5 | Logo, typeface and exact colour tokens | Direction set (blue, pastel, calm); finalise after the clickable prototype |
| D6 | Page structure for the calculator | **Decided: B** (after the prototype, 2026-11-14). Landing with intro + From → To + CTA, leading to a separate planner page with result, map and levers. The user is not overwhelmed at the start; the landing stays fast and map-free; the planner page is the share target. |
| D7 | What "No tracking" promises | **Decided** (2026-11-14): no analytics, no cookies, no third-party requests (fonts and map tiles self-hosted); only the hosting provider's request logs, described on the Privacy page with their retention |
