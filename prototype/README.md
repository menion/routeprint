# Routeprint UI prototype

Source of the clickable UI prototype (landing page + planner, desktop and mobile, light and dark),
built as a Claude Design canvas from [../docs/prototype-prompt.md](../docs/prototype-prompt.md).

Mock data only; decisions it led to are recorded in [../docs/design.md](../docs/design.md).

| Path | Content |
|---|---|
| [routeprint.css](routeprint.css) | Design tokens (CSS custom properties, light + dark) and components — the porting target for Astro |
| [canvas/Main.dc.html](canvas/Main.dc.html) | The whole UI as one component: markup, mock data, interaction logic |
| `canvas/B-*.dc.html` | Thin frames that render `Main` per page, width and theme |
| [canvas/canvas.json](canvas/canvas.json) | Canvas layout and notes |

- The `.dc.html` files use the Design canvas format (`<x-dc>`, `{{holes}}`, `<sc-if>`, `<sc-for>`,
  `DCLogic`) and only run inside the canvas; they are a reference, not production code.
- In the canvas the stylesheet is loaded from an uploaded asset (`/_blob/…` in each frame's
  `<head>`); this folder's `routeprint.css` is the same file.
- Inter is loaded from Google Fonts in the canvas only. Do not port that link: production
  self-hosts the font or uses the system stack, so "No tracking" holds (design.md D7).
- Values marked "example" in the UI (stop factor, per-aircraft factors, take-off/landing split)
  are placeholders, not sourced data.
