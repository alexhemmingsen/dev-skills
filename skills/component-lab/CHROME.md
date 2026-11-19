# The Lab Chrome

The chrome is harness, not design, and it looks the same on every run. It follows the Figma design `heapsgo-exploration-design`, node `447:6243`. Copy [lab.html](lab.html); this file describes what it does. Per run, only the component name, variants, global controls and scenarios change.

## Layout

White page, a thin top bar, then the grid and a controls panel, split by hairlines (`rgba(0,0,0,.08)`). Inter 12px throughout.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ Component lab · Buttons          Variant [Action ⌄] │ Background color [#FFF ▢]│
├──────────────────┬──────────────────┬──────────────────┬────────┬────────┤
│ ★ Interactive    │ default          │ loading          │ …      │ »      │
│   [  Save  ]     │   [  Save  ]     │   [ Saving… ]    │        │ Shape  │
├──────────────────┼──────────────────┼──────────────────┼────────┤ [R][E] │
│ disabled         │ hover            │ focus            │ …      │ Size   │
│   [  Save  ]     │   [  Save  ]     │   [  Save  ]     │        │ [S][M] │
└──────────────────┴──────────────────┴──────────────────┴────────┴────────┘
```

- **Top bar.** Left: `Component lab`, a dot, the component name in grey. Right: a **Variant** label and a native `<select>` (styled to the design: hairline border, 8px radius, chevron), shown only when the component has more than one variant, then a thin divider, then **Background color**: a label and a chip (hex value and swatch, a native color input kept in the URL as `bg`). Variants are what the thing *is*, not its states.
- **The page background.** The Background color chip sets the background of the grid area, the page the components sit on. It is the only thing that changes it. The top bar and the controls panel stay white.
- **The grid.** One area, no left column. **The first cell is the interactive component**, headed by a star and "Interactive": real hover, focus and typing, every action stubbed (see Stubbing), showing the variant chosen in the select. Every other cell is a **snapshot of one state of the selected variant**, headed by a small grey label (the scenario). Snapshots are not interactive: no hover, no clicks. Switching the variant redraws the grid with that variant's states, and a scenario can list the `variants` it applies to (a textarea's resize handle, a password's show toggle). Scenarios keep their group order.
  - **Any number of cells, always tidy.** There can be 1 cell or 40. Columns are as wide as the component needs (`CELL_WIDTH` for components that fill their container, like inputs and cards; the measured natural width otherwise), so nothing overflows or overlaps a neighbor. As many columns fit as the width allows, and the last row is filled with empty bordered cells so the grid is always a complete rectangle. Content sits top-left under the header, so headers and components line up across a row.
- **Right panel: global controls.** A white panel with one labelled segmented control per **global axis**, meaning anything that changes the look of *all* instances at once: theme, size, shape or rounding, density, width. A chevrons button at the top folds the panel. With no global axes, the panel is hidden.

The segmented control is the one control style: a rounded track (`rgba(0,0,0,.08)`), the selected option as a white pill with a hairline shadow.

## Stubbing

The preview component behaves as it does in the app, with no side effects. Replace every callback prop with a no-op, never call the network or write to a store, and feed it mock data from the scenarios. Pressing Delete deletes nothing.

## URL

Every control lives in the query string, written with `history.replaceState`: `?variant=<id>&shape=edgy&size=lg&theme=dark&bg=%23ff0000`. Missing params fall back to the first value of each axis. Load reads the URL once.

## Axes and theme

Global axes set `data-<key>` (`data-theme`, `data-size`, `data-shape`) on each component's slot, and pass the values to `render`. They style **the components only**. The theme never changes the page or the chrome: a dark theme makes the components dark on whatever background the Background color chip holds. Only the Background color chip changes the page background.

## Forced states

A scenario may carry `force` (`hover`, `focus`, `pressed`). The harness sets `data-force` on that cell's component, so the fork's twin selectors (`.btn:hover, .btn[data-force="hover"]`) render it with no mouse. The preview component never gets `data-force`. A scenario can combine data and force (`cancelled` + `hover`).

## Template

[lab.html](lab.html) is the complete page: markup, CSS and script in one self-contained file. Copy it, then edit only the block marked `Per run` (name, variants, axes, scenarios, `render`). The harness below that block stays as is.

In an app, the lab is a route (or a search-param guard, see Phase 3 of `SKILL.md`) inside the app, never a loose file. Port the template with the class names, CSS and structure unchanged, and express the rendering idiomatically: state instead of `innerHTML`, the fork imported as a normal component so hot reload updates every cell, and the CSS in a stylesheet or style block next to the route.
