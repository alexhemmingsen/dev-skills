# The Lab Chrome

The chrome is harness, not design, and it looks the same on every run. Copy [lab.html](lab.html); the spec below describes what it does. Only the component name, variants, axis toggles and scenarios change per run. It is a plain white page with thin grey dividers, so it reads as chrome on any project. It turns dark only when the component has a theme axis and the user picks dark.

## Layout

A white page, split in two, with a white top bar. Thin dividers separate the bar, the two halves and every grid cell.

```
┌───────────────────────────────────────────────────────────────────┐
│ Button  [Primary][Danger]            size [sm][md][lg]  theme […] │
├───────────────────┬───────────────────────────────────────────────┤
│ INTERACTIVE       │ Primary · default │ Primary · loading │ Primary · … │
│                   ├───────────────────┼───────────────────┼─────────────┤
│   ┌─────────┐     │      [ Save ]     │     [Saving…]     │   [ Save ]  │
│   │  Save   │     ├───────────────────┼───────────────────┼─────────────┤
│   └─────────┘     │ Danger · default  │ Danger · loading  │ Danger · …  │
│                   │     [Delete]      │    [Deleting]     │   [Delete]  │
└───────────────────┴───────────────────┴───────────────────┴─────────────┘
```

- **Top bar.** White. Component name, a variant switch (only if there is more than one variant), and one toggle group per other axis the component has (size, theme, width). Nothing for an axis the component lacks.
- **Left half: the interactive component.** One live instance, labelled "Interactive", centered. Real hover, focus and typing, with every action stubbed (see Stubbing). It shows the variant chosen in the variant switch.
- **Right half: the grid.** One cell per variant and scenario, divided by horizontal and vertical lines. Each cell has a small label (`variant · scenario`, or just `scenario` with one variant) above the real component at real size. Scenarios keep their group order, so related states sit together.

## Stubbing

The interactive component behaves as it does in the app, with no side effects. Replace every callback prop with a no-op (a toast or counter is fine to show it fired), never call the network or write to a store, and feed it mock data from the scenarios. Pressing Delete deletes nothing.

## URL

Axis choices live in the query string, written with `history.replaceState`: `?variant=<id>&size=lg&theme=dark`. Missing params fall back to the first value of each axis. Load reads the URL once.

## Forced states

A scenario may carry `force` (`hover`, `focus`, `pressed`). The harness sets `data-force` on that cell's component, so the fork's twin selectors (`.btn:hover, .btn[data-force="hover"]`) render it with no mouse. The interactive component never gets `data-force`. A scenario can combine data and force (`cancelled` + `hover`).

## Template

[lab.html](lab.html) is the complete page: markup, CSS and script in one self-contained file. Copy it, then edit only the block marked `Per run` (name, variants, axes, scenarios, `render`). The harness below that block stays as is.

In a framework, port it with the class names, CSS and structure unchanged, and express the rendering idiomatically.
