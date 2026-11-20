# Lab mechanics

Read from the Phase 3 steps of [SKILL.md](SKILL.md) when the component has overlays, and when mounting the lab.

## Overlays

Listboxes, menus, popovers, date pickers and tooltips mount inside the cell: point the component's portal container at `host`, the fifth argument of `render` and the cell's own element, or render the panel in flow. A panel portaled to `body` escapes the grid.

## Mount

- The guard returns the lab instead of the app when `?ahem-skill=<component>` is present, so no page, layout, effect, request or redirect underneath runs.
- Place it where the whole app is rendered: React in `main.tsx` or the root `App`; Next as a small client component rendered from the root layout, so fixtures cross no server boundary; the equivalent elsewhere.
- Put it inside the providers the component needs (theme, i18n, query client).
- The lab's root is a fixed full-viewport overlay. When it sits in a container with `transform`, `filter`, `backdrop-filter`, `will-change` or `contain`, render it through a portal to `body`.
