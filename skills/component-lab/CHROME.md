# The Lab Chrome

The chrome is harness, not design, and it looks the same on every run. It follows the Figma design `heapsgo-exploration-design`, node `447:6243`. Copy [lab.html](lab.html); this file describes what it does. Per run, only the component name, variants, global controls and scenarios change.

## Layout

White page, a thin top bar, then three columns split by hairlines (`rgba(0,0,0,.08)`). Inter 12px throughout.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ Component lab · Buttons                                                  │
├──────────────────────┬─────────────────────────────────────┬─────────────┤
│ Preview              │ Action · default │ Action · loading │ »           │
│ [Action][Neutral]    │                  │                  │ Shape       │
├──────────────────────┼──────────────────┼──────────────────│ [Round][Edgy]
│ ▢ Background color   │ Action · hover   │ Action · focus   │ Size        │
├──────────────────────┼──────────────────┼──────────────────│ [S][M][L]   │
│                      │ Neutral · default│ Neutral · loading│ Theme       │
│     [  Save  ]       │                  │                  │ [Light][Dark]
└──────────────────────┴──────────────────┴──────────────────┴─────────────┘
```

- **Top bar.** `Component lab`, a dot, then the component name in grey.
- **Left column: the preview.** Top: the **variant switch** under a "Preview" label, shown only when the component has more than one variant (variants are what the thing *is*, not its states). Then a **Background color** chip that sets the preview area's background (a native color input, kept in the URL as `bg`). Below: the **preview area**, with the live interactive component centered, every action stubbed (see Stubbing). It shows the variant chosen in the switch.
- **Middle: the grid.** A snapshot of every state of the **selected variant** only. Switching the variant redraws the grid with that variant's states, and a scenario can list the `variants` it applies to (a textarea's resize handle, a password's show toggle). Each cell has a small grey label (the scenario) above the real component. Cells are divided by horizontal and vertical lines and are not interactive: no hover, no clicks. Scenarios keep their group order.
  - **Any number of cells, always tidy.** There can be 1 cell or 40. Columns are as wide as the component needs (`CELL_WIDTH` for components that fill their container, like inputs and cards; the measured natural width otherwise), so nothing overflows or overlaps a neighbor. As many columns fit as the width allows, and the last row is filled with empty bordered cells so the grid is always a complete rectangle. Content sits top-left under the label, so labels and components line up across a row.
- **Right column: global controls.** A light grey panel (`#fafafa`) with one labelled segmented control per **global axis**, meaning anything that changes the look of *all* instances at once: theme, size, shape or rounding, density, width. A chevrons button at the top folds the panel. With no global axes, the panel is hidden.

The segmented control is the one control style: a rounded track (`rgba(0,0,0,.08)`), the selected option as a white pill with a hairline shadow.

## Stubbing

The preview component behaves as it does in the app, with no side effects. Replace every callback prop with a no-op, never call the network or write to a store, and feed it mock data from the scenarios. Pressing Delete deletes nothing.

## URL

Every control lives in the query string, written with `history.replaceState`: `?variant=<id>&shape=edgy&size=lg&theme=dark&bg=%23ff0000`. Missing params fall back to the first value of each axis. Load reads the URL once.

## Axes and theme

Global axes set `data-<key>` (`data-theme`, `data-size`, `data-shape`) on the preview area and on every grid cell, and pass the values to `render`. The chrome itself never changes with them. `data-theme="dark"` turns the preview area and the cells dark (`#141416`), unless the user picked a background color for the preview.

## Forced states

A scenario may carry `force` (`hover`, `focus`, `pressed`). The harness sets `data-force` on that cell's component, so the fork's twin selectors (`.btn:hover, .btn[data-force="hover"]`) render it with no mouse. The preview component never gets `data-force`. A scenario can combine data and force (`cancelled` + `hover`).

## Template

[lab.html](lab.html) is the complete page: markup, CSS and script in one self-contained file. Copy it, then edit only the block marked `Per run` (name, variants, axes, scenarios, `render`). The harness below that block stays as is.

In an app, the lab is a route in the app's own router (see Phase 3 of `SKILL.md`), never a loose file. Port the template with the class names, CSS and structure unchanged, and express the rendering idiomatically: state instead of `innerHTML`, the fork imported as a normal component so hot reload updates every cell, and the CSS in a stylesheet or style block next to the route.
