# Routeprint

Transparent travel carbon footprint calculator: every number is traced to a cited source. Flights
come first; rail, road and a map-based trip planner will follow.

**Status:** design phase, no code yet.

## Why

Flight footprint calculators disagree by a factor of 2.5 or more for the same trip, mostly because
of hidden assumptions about non-CO2 effects, distance and allocation. Routeprint shows the range,
puts those assumptions on the surface as switchable, cited parameters, and explains every step of
the calculation. No offsets, no affiliate links.

## Documentation

| Document | Content |
|---|---|
| [Landscape review](docs/flight-footprint-landscape.md) | Existing calculators, why they diverge, where Routeprint fits |
| [Architecture](docs/architecture.md) | Engine, methodology as data, reference data, growth path, decisions |
| [Design](docs/design.md) | Audience, principles, core flow, explain view, open design questions |
| [Prototype prompt](docs/prototype-prompt.md) | Brief for a design tool to produce the first clickable UI prototype |

## Licence

- Code: [Apache-2.0](LICENSE)
- Documentation and own text content: [CC BY 4.0](LICENSES/CC-BY-4.0.txt)
- Data files derived from Google Travel Impact Model API figures:
  [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) (licence text added with the first such file)

Per-path licensing follows the [REUSE](https://reuse.software/) convention; see
[REUSE.toml](REUSE.toml).
