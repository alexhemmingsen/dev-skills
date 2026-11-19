# The Lab Chrome

The chrome is harness, not design, and it looks the same on every run. It follows the Figma design `heapsgo-exploration-design`, node `447:6243`. Copy [lab.html](lab.html); this file describes what it does. Per run, only the component name, variants, global controls and scenarios change.

## Layout

A page (white by default), a thin top bar, then the grid and a controls panel, split by hairlines (`rgba(0,0,0,.08)`). Inter 12px throughout.

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

- **Top bar.** 16px padding, every control 26px high with a hairline border and 8px radius. Left: `Component lab`, a dot, the component name in grey. Right, in groups 16px apart: a **Variant** label and a native `<select>` (shown only when the component has more than one variant; 8px between label and select), a 1px by 16px divider, then **Background color**: a label (8px gap), the chip (hex value, then a swatch) and, 4px beside it, the **project color variable** `<select>`. Variants are what the thing *is*, not its states.
- **Project color variable select.** A native `<select>` beside the chip, with a "Variable" placeholder first and then **every color variable in the project whose value is a raw color** (hex, `rgb()`, `hsl()`, `oklch()`, `color()`), listed by name (`--color-background-base`), with the value as the option's tooltip. Aliases (`var(--other)`, `calc()`) and non-color variables are left out. Picking one fills the page background with its value; the chip then shows the resolved hex. Choosing a manual color resets the select to "Variable". The choice is kept in the URL as `token`. With no raw color variables the select is hidden.
- **The page background.** The Background color chip sets the background of the **whole page**: top bar, grid and controls panel. Everything the lab draws itself follows it in a contrast color: the breadcrumb, labels, icons, hairlines, select, chip and segmented controls turn white on a dark background and black on a light one (picked by WCAG luminance), and the chrome's native controls (select, color input) switch with `color-scheme`, which is set on the chrome only so it never restyles a component's own native elements. The components keep their own colors: only text that is not part of a component adapts.
- **Background color picker.** The chip opens a popover with a saturation and brightness field, a hue slider, a hex field (3 or 6 digits, with or without `#`), an eyedropper where the browser has one, and a row of preset swatches (neutrals, near-black, soft tints). Dragging updates the whole page live. The popover keeps its own surface, so it never blends into the page color, and it closes on Escape or a click outside. The field and the slider also take arrow keys.
- **The interactive cell is gold.** The star and the "Interactive" label are gold, so it is obvious which cell is the live preview. The lab picks the gold that reads best on the page background (dark gold on light pages, bright gold on dark ones); on backgrounds where neither is readable, the label gets its own small gold pill.
- **The grid.** One area, no left column. **The first cell is the interactive component**, headed by a star and "Interactive": real hover, focus and typing, every action stubbed (see Stubbing), showing the variant chosen in the select. Every other cell is a **snapshot of one state of the selected variant**, headed by a small grey label (the scenario). Snapshots are not interactive: no hover, no clicks. Switching the variant redraws the grid with that variant's states, and a scenario can list the `variants` it applies to (a textarea's resize handle, a password's show toggle). Scenarios keep their group order.
  - **Nothing overflows a cell.** No widths are configured. A component that fills its container fills its slot (capped at a typical form-column width, 360px); one with its own size keeps it. After rendering, the grid measures how far each component really reaches (wide children, absolutely positioned menus and popups included), widens the columns to the widest and heightens a row where something hangs below, and scrolls horizontally when the window is narrower. It re-measures after the user clicks or types in the interactive cell, since that component may grow. A specific width belongs in the fork's own CSS, never in the lab, and overflow is never hidden by clipping.
  - **Any number of cells, always tidy.** There can be 1 cell or 40. As many columns fit as the width allows, and the last row is filled with empty bordered cells so the grid is always a complete rectangle. Content sits top-left under the header, so headers and components line up across a row.
- **Right panel: global controls.** A white panel with one labelled segmented control per **global axis**, meaning anything that changes the look of *all* instances at once: theme, size, shape or rounding, density, width. The panel is at least 150px wide when expanded, and grows with its widest control. A chevrons button at the top folds it to just that button. With no global axes, the panel is hidden.

The segmented control is the one control style: a rounded track (`rgba(0,0,0,.08)`), the selected option as a white pill with a hairline shadow.

## Stubbing

The preview component behaves as it does in the app, with no side effects. Replace every callback prop with a no-op, never call the network or write to a store, and feed it mock data from the scenarios. Pressing Delete deletes nothing.

## No backdrop

The component sits directly on the page. Every cell, and the interactive one, shows the component alone: its own fill and nothing around it (no sidebar surface, panel, card or wrapper with a background). The page color comes from the Background color control, so a backdrop behind a component would hide the very thing the user is changing.

## URL

Every control lives in the query string, written with `history.replaceState`: `?variant=<id>&shape=edgy&size=lg&theme=dark&bg=%23ff0000`. Missing params fall back to the first value of each axis. Load reads the URL once.

## Axes and theme

Global axes set `data-<key>` (`data-theme`, `data-size`, `data-shape`) on each component's slot, and pass the values to `render`. They style **the components only**. The theme never changes the page or the chrome: a dark theme makes the components dark on whatever background the Background color chip holds. Only the Background color chip changes the page, and the chrome follows it.

## Forced states

A scenario may carry `force` (`hover`, `focus`, `pressed`). The harness sets `data-force` on that cell's component, so the fork's twin selectors (`.btn:hover, .btn[data-force="hover"]`) render it with no mouse. The preview component never gets `data-force`. A scenario can combine data and force (`cancelled` + `hover`).

## Template

[lab.html](lab.html) is the complete page: markup, CSS and script in one self-contained file. Copy it, then edit only the block marked `Per run` (name, variants, axes, scenarios, `render`). The harness below that block stays as is.

The template's root (`.lab`) is `position: fixed; inset: 0`, so the lab fills the whole viewport and the controls panel and its divider run the full height, whatever layout, nav or containers the app wraps around the route. Keep that when porting; never size the lab with `height: 100%` on `html`, `body` or an app container.

In an app, the lab is a route (or a search-param guard, see Phase 3 of `SKILL.md`) inside the app, never a loose file. Port the template with the class names, CSS and structure unchanged, and express the rendering idiomatically: state instead of `innerHTML`, the fork imported as a normal component so hot reload updates every cell, and the CSS in a stylesheet or style block next to the route.
