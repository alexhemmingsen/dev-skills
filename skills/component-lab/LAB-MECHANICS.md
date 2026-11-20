# Lab mechanics

Read from the Phase 3 steps of [SKILL.md](SKILL.md) when the component has overlays, and when mounting the lab.

## Overlays

Listboxes, menus, popovers, date pickers and tooltips mount inside the cell: point the component's portal container at `host`, the fifth argument of `render` and the cell's own element, or render the panel in flow. A panel portaled to `body` escapes the grid. Every cell is live, so a panel a user opens in one cell may overlap the cells around it; it closes again, and the grid stays still.

Overlays, and everything else that appears during interaction, carry every global axis. The lab sets each axis on the cell's slot as `data-<axis>`, which reaches only its descendants: a dialog portaled to `body` lies outside it and misses every axis alike. Two ways to reach it:

1. Mount overlays inside `host`, where they inherit the slot.
2. Supply each axis at the root of `render` the way production does (the provider or context that holds it, such as theme, locale, density or size, or the class or attribute the CSS keys on), around the component and its portal container, so whatever mounts later is under it. Use this when the overlay library cannot take a container, or when an axis lives in context.

Then interact in each cell to open every dialog, menu, popover and toast with each axis switched, and check each one follows.

## Mount

- The guard returns the lab instead of the app when `?ahem-skill=<component>` is present, so no page, layout, effect, request or redirect underneath runs.
- Place it where the whole app is rendered: React in `main.tsx` or the root `App`; Next as a small client component rendered from the root layout, so fixtures cross no server boundary; the equivalent elsewhere.
- Put it inside the providers the component needs (theme, i18n, query client).
- The lab's root is a fixed full-viewport overlay. When it sits in a container with `transform`, `filter`, `backdrop-filter`, `will-change` or `contain`, render it through a portal to `body`.
