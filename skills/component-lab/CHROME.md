# The Lab Chrome

The chrome is harness, not design. Copy the markup, CSS and wiring below. Only the component name, variants, axis toggles, groups and scenarios change per run. It is dark and not theme-aware, so it reads as chrome on any project. Only the **stage** takes the project's background.

## Layout

```
┌───────────────────────────────────────────────────────────────────┐
│ Button  [Primary][Danger]            size [sm][md][lg]  theme […] │
├───────────────────┬───────────────────────────────────────────────┤
│ INTERACTIVE       │ DATA                                          │
│                   │            default  loading  disabled         │
│   ┌─────────┐     │  Primary   [ Save ] [Saving…] [ Save ]        │
│   │  Save   │     │  Danger    [Delete] [Deleting] [Delete]       │
│   └─────────┘     │ INTERACTION                                   │
│                   │            hover    focus    pressed          │
│                   │  Primary   [ Save ] [ Save ] [ Save ]         │
└───────────────────┴───────────────────────────────────────────────┘
```

- **Top bar.** Component name, a variant switch (only if there is more than one variant), and one toggle group per other axis the component has (size, theme, width). Nothing for an axis the component lacks.
- **Left: the interactive component.** One live instance, labelled "Interactive". Real hover, focus and typing, with every action stubbed (see Stubbing). It shows the variant chosen in the variant switch.
- **Right: every state at once.** A grid, one cell per scenario, each with a small label under a group heading. With more than one variant, variants are rows and scenarios are columns. Cells are real components at real size, so a change shows across every state in one view.
- **Stage.** Both panes sit on the project's page background, in realistic surroundings (a button in a form, a card beside sibling cards).

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
  :root { --lab-bg: #0b0b0c; --lab-fg: #e8e8ea; --lab-dim: #8b8b92; --lab-line: rgba(255,255,255,.08); --lab-hi: rgba(255,255,255,.1); }
  * { box-sizing: border-box; }
  html, body { height: 100%; margin: 0; }
  body { display: grid; grid-template-rows: auto 1fr; font: 13px/1.4 -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; background: var(--lab-bg); color: var(--lab-fg); }
  .lab-bar { display: flex; flex-wrap: wrap; align-items: center; gap: 8px 20px; padding: 10px 16px; border-bottom: 1px solid var(--lab-line); }
  .lab-name { font-weight: 600; }
  .lab-spacer { flex: 1; }
  .lab-group { display: flex; align-items: center; gap: 2px; padding: 2px; border-radius: 8px; background: var(--lab-hi); }
  .lab-group > span { padding: 0 8px; color: var(--lab-dim); }
  .lab-btn { padding: 4px 10px; border: 0; border-radius: 6px; background: none; color: inherit; font: inherit; cursor: pointer; }
  .lab-btn:hover { background: var(--lab-hi); }
  .lab-btn[aria-pressed="true"] { background: rgba(255,255,255,.18); font-weight: 600; }
  .lab-body { display: grid; grid-template-columns: minmax(280px, 1fr) 3fr; min-height: 0; }
  .lab-stage { --stage-bg: #fff; --stage-fg: #111; min-width: 0; overflow: auto; padding: 20px; background: var(--stage-bg); color: var(--stage-fg); }
  .lab-stage[data-theme="dark"] { --stage-bg: #141416; --stage-fg: #eee; }
  .lab-left { display: grid; place-content: center; justify-items: start; gap: 12px; border-right: 1px solid var(--lab-line); }
  .lab-cap { font-size: 11px; letter-spacing: .04em; color: #888; }
  .lab-h { margin: 20px 0 8px; font-size: 11px; font-weight: 600; letter-spacing: .06em; text-transform: uppercase; color: #888; }
  .lab-h:first-child { margin-top: 0; }
  .lab-grid { display: grid; gap: 14px 20px; align-items: center; justify-items: start; }
  .lab-wrap { display: flex; flex-wrap: wrap; gap: 14px 20px; }
  .lab-cell { display: grid; gap: 6px; justify-items: start; }
</style>
</head>
<body>
<header class="lab-bar" id="bar"></header>
<div class="lab-body">
  <section class="lab-stage lab-left" id="left"></section>
  <section class="lab-stage" id="right"></section>
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
function stage(node) { Object.entries(axes()).forEach(([k, v]) => node.dataset[k] = v); }
function drawBar() {
  const bar = $('#bar'); bar.replaceChildren(el('span', 'lab-name', NAME));
  if (VARIANTS.length > 1) bar.append(toggle('', 'variant', VARIANTS));
  bar.append(el('span', 'lab-spacer'));
  AXES.forEach((a) => bar.append(toggle(a.key, a.key, a.values.map((v) => ({ id: v, label: v })))));
}
function drawLeft() {
  const l = $('#left'); stage(l); l.replaceChildren(el('div', 'lab-cap', 'INTERACTIVE'));
  l.append(render(st.variant, (SCENARIOS[0] || {}).props || {}, '', axes()));
}
function drawRight() {
  const r = $('#right'); stage(r); r.replaceChildren();
  const groups = [...new Set(SCENARIOS.map((s) => s.group))];
  groups.forEach((g) => {
    r.append(el('h3', 'lab-h', g));
    const list = SCENARIOS.filter((s) => s.group === g);
    if (VARIANTS.length > 1) {
      const grid = el('div', 'lab-grid'); grid.style.gridTemplateColumns = `auto repeat(${list.length}, auto)`;
      grid.append(el('span')); list.forEach((s) => grid.append(el('span', 'lab-cap', s.label)));
      VARIANTS.forEach((v) => { grid.append(el('span', 'lab-cap', v.label)); list.forEach((s) => grid.append(render(v.id, s.props, s.force, axes()))); });
      r.append(grid);
    } else {
      const wrap = el('div', 'lab-wrap');
      list.forEach((s) => { const c = el('div', 'lab-cell'); c.append(render(undefined, s.props, s.force, axes()), el('span', 'lab-cap', s.label)); wrap.append(c); });
      r.append(wrap);
    }
  });
}
function draw() { drawBar(); drawLeft(); drawRight(); }
draw();
</script>
</body>
</html>
```
