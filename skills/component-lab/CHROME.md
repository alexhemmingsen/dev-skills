# The Lab Chrome

The chrome is harness, not design. Copy the markup, CSS and wiring below. Only the component name, variants, axis toggles and scenarios change per run. It is a plain white page with thin grey dividers, so it reads as chrome on any project. It turns dark only when the component has a theme axis and the user picks dark.

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

## Reference implementation

Self-contained HTML. In a framework, keep the class names, structure and behavior and express the rendering idiomatically.

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Component Lab</title>
<style>
  :root { --bg: #fff; --fg: #111; --dim: #8a8a8f; --line: #e6e6e8; --hi: #f1f1f3; }
  :root[data-theme="dark"] { --bg: #141416; --fg: #eee; --dim: #8b8b92; --line: #2a2a2e; --hi: #232327; }
  * { box-sizing: border-box; }
  html, body { height: 100%; margin: 0; }
  body { display: grid; grid-template-rows: auto 1fr; font: 13px/1.4 -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; background: var(--bg); color: var(--fg); }
  .lab-bar { display: flex; flex-wrap: wrap; align-items: center; gap: 8px 20px; padding: 10px 16px; border-bottom: 1px solid var(--line); }
  .lab-name { font-weight: 600; }
  .lab-spacer { flex: 1; }
  .lab-group { display: flex; align-items: center; gap: 2px; padding: 2px; border: 1px solid var(--line); border-radius: 8px; }
  .lab-group > span { padding: 0 8px; color: var(--dim); }
  .lab-btn { padding: 4px 10px; border: 0; border-radius: 6px; background: none; color: inherit; font: inherit; cursor: pointer; }
  .lab-btn:hover { background: var(--hi); }
  .lab-btn[aria-pressed="true"] { background: var(--hi); font-weight: 600; }
  .lab-body { display: grid; grid-template-columns: minmax(280px, 1fr) 3fr; min-height: 0; }
  .lab-left { position: relative; display: grid; place-content: center; padding: 20px; border-right: 1px solid var(--line); }
  .lab-right { overflow: auto; }
  .lab-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); border-top: 0; }
  .lab-cell { display: grid; grid-template-rows: auto 1fr; gap: 14px; padding: 12px 14px; border-right: 1px solid var(--line); border-bottom: 1px solid var(--line); }
  .lab-cell > :last-child { place-self: center; }
  .lab-cap { font-size: 11px; color: var(--dim); }
  .lab-left .lab-cap { position: absolute; top: 12px; left: 14px; }
</style>
</head>
<body>
<header class="lab-bar" id="bar"></header>
<div class="lab-body">
  <section class="lab-left" id="left"></section>
  <section class="lab-right"><div class="lab-grid" id="right"></div></section>
</div>
<script>
// ---- Per run: replace these ----
const NAME = 'Button';
const VARIANTS = [{ id: 'primary', label: 'Primary' }, { id: 'danger', label: 'Danger' }]; // [] when there are none
const AXES = [{ key: 'size', values: ['sm', 'md', 'lg'] }, { key: 'theme', values: ['light', 'dark'] }]; // only axes the component has
const SCENARIOS = [ // { id, label, group, props, force? }
  { id: 'default', label: 'default', group: 'Data', props: {} },
  { id: 'loading', label: 'loading', group: 'Data', props: { loading: true } },
  { id: 'disabled', label: 'disabled', group: 'Data', props: { disabled: true } },
  { id: 'hover', label: 'hover', group: 'Interaction', props: {}, force: 'hover' },
  { id: 'focus', label: 'focus', group: 'Interaction', props: {}, force: 'focus' },
  { id: 'pressed', label: 'pressed', group: 'Interaction', props: {}, force: 'pressed' },
];
// render(variant, props, force, axes) returns an element: the forked component with every action stubbed.
// Set data-force when force is given. Read axes (size, theme, ...) however the component does.
function render(variant, props, force, axes) {
  const el = document.createElement('button');
  el.textContent = props.loading ? 'Saving…' : variant === 'danger' ? 'Delete' : 'Save';
  el.disabled = !!props.disabled;
  el.dataset.variant = variant; el.dataset.size = axes.size;
  if (force) el.dataset.force = force;
  el.onclick = () => {}; // stub: nothing happens
  return el;
}
// ---- Harness: leave as is ----
const $ = (s) => document.querySelector(s);
const q = new URLSearchParams(location.search);
const st = { variant: q.get('variant') || (VARIANTS[0] && VARIANTS[0].id) };
AXES.forEach((a) => { st[a.key] = a.values.includes(q.get(a.key)) ? q.get(a.key) : a.values[0]; });
const axes = () => Object.fromEntries(AXES.map((a) => [a.key, st[a.key]]));
function el(tag, cls, text) { const e = document.createElement(tag); if (cls) e.className = cls; if (text != null) e.textContent = text; return e; }
function sync() { const u = new URL(location); ['variant', ...AXES.map((a) => a.key)].forEach((k) => st[k] ? u.searchParams.set(k, st[k]) : u.searchParams.delete(k)); history.replaceState(null, '', u); }
function pick(k, v) { st[k] = v; sync(); draw(); }
function toggle(label, key, options) { const g = el('div', 'lab-group'); if (label) g.append(el('span', '', label)); options.forEach((o) => { const b = el('button', 'lab-btn', o.label); b.setAttribute('aria-pressed', String(st[key] === o.id)); b.onclick = () => pick(key, o.id); g.append(b); }); return g; }
function theme() { Object.entries(axes()).forEach(([k, v]) => document.documentElement.dataset[k] = v); }
function drawBar() {
  const bar = $('#bar'); bar.replaceChildren(el('span', 'lab-name', NAME));
  if (VARIANTS.length > 1) bar.append(toggle('', 'variant', VARIANTS));
  bar.append(el('span', 'lab-spacer'));
  AXES.forEach((a) => bar.append(toggle(a.key, a.key, a.values.map((v) => ({ id: v, label: v })))));
}
function drawLeft() {
  const l = $('#left'); l.replaceChildren(el('div', 'lab-cap', 'Interactive'));
  l.append(render(st.variant, (SCENARIOS[0] || {}).props || {}, '', axes()));
}
function drawRight() {
  const r = $('#right'); r.replaceChildren();
  const rows = VARIANTS.length > 1 ? VARIANTS : [{ id: VARIANTS[0] && VARIANTS[0].id, label: '' }];
  rows.forEach((v) => SCENARIOS.forEach((s) => {
    const c = el('div', 'lab-cell'); c.append(el('span', 'lab-cap', v.label ? `${v.label} · ${s.label}` : s.label), render(v.id, s.props, s.force, axes())); r.append(c);
  }));
}
function draw() { theme(); drawBar(); drawLeft(); drawRight(); }
draw();
</script>
</body>
</html>
```
