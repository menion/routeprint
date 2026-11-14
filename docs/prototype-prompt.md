# Routeprint — UI prototype prompt

Historical brief that produced the first clickable UI prototype (2026-11-14).

Where it differs from [design.md](design.md), design.md wins: D6 chose variant B, D1 made one way
the default, D4 shortened the switch labels. To reuse it, copy everything below the line.

---

## Task

Design a clickable UI prototype for **Routeprint**, a website that tells everyday travellers the
climate impact of a flight in plain words, and lets anyone who wants to dig deeper see exactly how
the number was calculated. Use static mock data; no real calculation or backend.

Deliver **two variants of the page structure** so they can be compared side by side (a visible
switch "Variant A / Variant B" in the prototype is fine):

- **Variant A — single page:** intro, From → To input, result, map and levers all on one page.
- **Variant B — two pages:** a light landing page (short intro + From → To input + CTA) that leads
  to a separate planner page with the result, map and levers.

Show each variant on **mobile (375 px) and desktop (1280 px)**, in **light and dark** theme.

## Product in one paragraph

Flight footprint calculators disagree by 2.5× for the same trip because they hide their
assumptions. Routeprint gives one honest headline number with a range, shows the main assumption
(climate effects of contrails and other high-altitude emissions) as a simple visible switch, tells
the user what would change the number (cabin class, connections, aircraft), and keeps every detail
one tap away behind "?" icons and "Read more". No offsets, no ads, no guilt.

## Audience and voice

- Primary: everyday travellers, not scientists. Plain language, short sentences, no jargon on the
  first layers.
- Secondary: curious or sceptical readers, journalists — they open the details.
- Voice: calm, factual, friendly. Never alarmist, never preachy, never green-washing.

## Screens and states to include

1. **Landing** (variant B) / top of the single page (variant A): product name, one-line promise,
   From and To airport fields, trip type (Return / One way), cabin, passengers, CTA "See the
   impact". Small trust line: "Open methodology · No offsets · No tracking". Below the fields, a
   "More precise: choose aircraft" link revealing an optional aircraft select (default "Typical
   mix on this route"); per-aircraft values are marked "example values" with a "?" popover.
2. **Airport autocomplete:** typing "Prag" shows "Prague (PRG) — Václav Havel Airport, Czechia";
   accepts city names, airport names and codes; keyboard-navigable list.
3. **Result card:**
   - Headline: **1.3 t CO2e** per passenger, return. Range below: "0.84 – 2.2 t depending on how
     contrail and high-altitude effects are counted".
   - Assumption switch, three segments in plain words: "CO2 only" · "Including high-altitude
     effects (recommended)" · "High estimate". Each with a "?" icon.
   - Breakdown bar with three labelled parts: Fuel burned (694 kg) · Fuel production (141 kg) ·
     Contrails and other high-altitude effects (486 kg).
   - Precision note: "Aircraft: typical mix on this route. Choose a specific flight for more
     precision."
   - Actions: "How is this calculated?" (Read more) and "Share".
4. **"?" popovers — on every number.** The headline, the range, each breakdown part, each lever
   delta and each comparison has a small "?" icon. Every popover follows one pattern:
   - one or two plain-language sentences: what the number is and why;
   - the derivation in one line (the values compared or multiplied);
   - source chip(s), e.g. "Google TIM 2026", "EEA 2023";
   - where available, a cross-check line, e.g. "Cross-check: Google TIM 418 kg — matches";
   - a "Read more" link to the explain view.

   Example, high-altitude effects segment: "Contrails and nitrogen oxides from aircraft at cruise
   altitude warm the climate in addition to CO2. We count them as 0.7× the CO2 from burned fuel
   (694 kg × 0.7 = 486 kg). Source: UK Government factors; Lee et al. 2021."
5. **Explain view (expanded "Read more"):** a step list — Distance 6,550 km (great-circle) → +5%
   real-route correction → fuel for take-off and landing (fixed per flight) → fuel for climb,
   cruise and descent (by distance) → fuel per passenger → CO2 from fuel → fuel production →
   high-altitude effects ×1.7 → total. Each step with its value and a short source name ("EEA
   2023", "Teoh et al."). Under the steps, a small panel "Why short flights burn more fuel per
   kilometre" with a three-row table for a Boeing 787-9 (926 km: 8.1 kg/km · 1,852 km: 6.8 kg/km
   · 9,260 km: 5.9 kg/km; source: EEA 2023 via Google TIM documentation). A small "Methodology
   2026.1" badge. Link "Full methodology and source code on GitHub".
6. **Map:** calm, low-contrast base map, great-circle arc Prague → New York with two airport
   markers. In variant B it loads after the result (show a light skeleton placeholder first).
7. **What changes this** (levers), each as a row with delta, a "?" popover proving it, and a
   chevron:
   - Business class: **+240%** — popover: "A business seat takes several times the cabin area of
     an economy seat, so it carries more of the flight's emissions. Google TIM for this route:
     business 1,434 kg vs. economy 417.5 kg per passenger, one way (×3.43). Source: Google TIM
     2026."
   - Premium economy: **+39%** — popover: "Google TIM for this route: premium economy 579.6 kg
     vs. economy 417.5 kg (×1.39). Source: Google TIM 2026."
   - One connection via Frankfurt: **+8%** — popover with distance and extra take-off/landing
     explanation (values are mock; mark them "example").
   - Newest aircraft type on this route: **−15%** — popover comparing two aircraft fuel burns
     (values are mock; mark them "example").
8. **In context:** a simple horizontal bar comparing this trip with two reference points, labelled
   as placeholders, e.g. "Reference A (source to be decided)". Do not invent real statistics.
9. **Share:** desktop shows "Link copied" confirmation; mobile shows the native share sheet
   trigger. Also show the calm notice for an older shared result: "Calculated with methodology
   2026.1. With the current version: 1.2 t (−8%). Show current."
10. **Error state:** unknown airport code — field keeps the input, inline message, rest of the form
    stays filled.
11. **Header and footer:** logo wordmark "Routeprint", navigation: Calculator · How it works ·
    Learn · About; footer: methodology, sources, GitHub, licence, "Support Routeprint" (same
    weight as the other links). Language switcher present but showing only "EN".
12. **Support line** at the end of the explain view only: "This calculation is free and open. If it
    helped you, you can support the work." — quiet text link, not a button.

All numbers above are mock data for Prague ⇄ New York JFK, economy, one passenger.

## Visual direction

- **Modern but fast:** clean and contemporary, with conservative, compact spacing —
  information-dense but uncluttered, not airy. Subtle depth (soft shadows or borders), no large
  photos, no heavy illustrations.
- **Animation:** a few small JavaScript animations are welcome but not required — e.g. the
  headline number counting up, breakdown bar segments filling, the map arc drawing, the
  assumption switch sliding. Keep them short (< 400 ms), sparse, and disabled under
  `prefers-reduced-motion`.
- **Colours:** blue-led, pastel, not too bright. Soft desaturated blues (powder, mist, steel) for
  surfaces and fills; a deeper muted blue for primary actions and links. Breakdown categories: two
  blues plus a muted sand or sage, colour-blind safe. No red for results; this is information, not
  an alarm.
- **Contrast:** pastels only for backgrounds, cards and chart fills. Text and controls meet WCAG AA
  (4.5:1 body text) in both themes.
- **Dark theme:** deep desaturated navy background, the same pastels slightly dimmed.
- **Typography:** one family (system stack or a single variable font), tabular figures for all
  numbers, a large confident headline number.
- **Calm editorial feel**, closer to a good data-journalism piece than to a campaign site.

## Constraints

- Mobile first; everything usable with keyboard and screen reader (visible focus, labels, ARIA for
  the switch and popovers).
- Built to be fast: aim for a landing page with almost no JavaScript; the map is lazy-loaded.
- Ready for translation: no text in images, layouts tolerate 30–40% longer strings, use logical
  CSS properties.
- Output as plain HTML + CSS with minimal vanilla JS, design tokens as CSS custom properties
  (colours, spacing, radii, type scale) for both themes, so it can be ported to Astro components.

## Avoid

- Offsets, "compensate now" or donation buttons, affiliate-style booking buttons. The only support
  links are the quiet ones in the footer and at the end of the explain view — never near the
  result or the share button.
- Guilt messaging, red alarm colours, sad-planet imagery, emoji.
- Jargon on the first layers ("RFI", "ERF", "WTW", "TTW").
- Fake precision (no decimals on headline numbers).
- Invented statistics in comparisons.
- Any number on screen without a "?" reference.
