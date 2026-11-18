# The Lab Chrome

The chrome is harness, not design. Copy the markup, CSS and wiring below. Only the component name, variant tabs, axis toggles, sidebar groups and scenarios change per run. It is dark and not theme-aware, so it reads as chrome on any project. Only the **stage** takes the project's background.

## Layout

```
┌───────────────┬───────────────────────────────────────────────┐
│ Component     │ Button   [Primary][Secondary][Danger]  Force: │
│ Overview      │                              [hover][focus]…  │
│ ───────────── ├───────────────────────────────────────────────┤
│ DATA          │                                               │
│  default      │                    stage                      │
│  loading      │            (one scenario, full size)          │
│ INTERACTION   │                                               │
│  hover        │                                               │
└───────────────┴───────────────────────────────────────────────┘
```

- **Sidebar, left, 240px.** "Component" and "Overview" on top, then one group heading per discovered group, one item per scenario.
- **Top bar.** Component name, variant tabs (only if there is more than one variant), toggles for other axes that exist (size, theme, width), and a **Force** row (`hover`, `focus`, `pressed`) when the component has pseudo-states.
- **Stage.** One scenario at full size on the project's page background, in realistic surroundings: a button inside a form, a card beside sibling cards.

## Views

| View | Shows |
| --- | --- |
| **Component** (default) | One live instance, fully interactive, real hover and focus, the default scenario |
| **Overview** | Grid: variants down the side, scenarios across the top. Every cell is the real component at real size. Clicking a cell opens it |
| **A scenario** | That scenario in the stage |

## URL

State lives in the query string and is written with `history.replaceState`:

`?state=<id|overview>&variant=<id>&force=<hover|focus|pressed>&size=lg&theme=dark&width=375`

A missing `state` means the Component tab. Missing axis params fall back to the first value of each axis. Load reads the URL once, with no animation.

## Behavior

- Click a sidebar item or variant tab to switch. Switching re-mounts the stage instantly.
- `↑` `↓` step through sidebar items, `←` `→` step through variant tabs. Ignore keys when focus is in an input, textarea, select or contenteditable, or when a modifier is held.
- `force` sets `data-force` on the component root. It overlays any scenario, so `?state=cancelled&force=hover` shows a hovered cancelled card.
- Exactly one sidebar item carries `aria-current="true"`.

## Reference implementation

Self-contained HTML. In a framework, keep the class names, structure and behavior and express the rendering idiomatically (state instead of `innerHTML`, a keyed re-mount instead of rebuilding).

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Component Lab</title>
<style>
  :root { --lab-bg: #0b0b0c; --lab-fg: #e8e8ea; --lab-dim: #8b8b92; --lab-line: rgba(255,255,255,.08); --lab-hi: rgba(255,255,255,.1); --stage-bg: #fff; }
  * { box-sizing: border-box; }
  html, body { height: 100%; margin: 0; }
  body { display: grid; grid-template-columns: 240px 1fr; font: 13px/1.4 -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; background: var(--lab-bg); color: var(--lab-fg); }
  .lab-side { padding: 12px 8px; overflow-y: auto; border-right: 1px solid var(--lab-line); }
  .lab-side h2 { margin: 16px 8px 4px; font-size: 11px; font-weight: 600; letter-spacing: .06em; text-transform: uppercase; color: var(--lab-dim); }
  .lab-item { display: block; width: 100%; padding: 6px 8px; border: 0; border-radius: 6px; background: none; color: inherit; font: inherit; text-align: left; cursor: pointer; }
  .lab-item:hover { background: var(--lab-hi); }
  .lab-item[aria-current="true"] { background: var(--lab-hi); font-weight: 600; }
  .lab-rule { height: 1px; margin: 8px; background: var(--lab-line); }
  .lab-main { display: grid; grid-template-rows: auto 1fr; min-width: 0; }
  .lab-bar { display: flex; flex-wrap: wrap; align-items: center; gap: 8px 16px; padding: 10px 16px; border-bottom: 1px solid var(--lab-line); }
  .lab-name { font-weight: 600; margin-right: 8px; }
  .lab-group { display: flex; gap: 2px; padding: 2px; border-radius: 8px; background: var(--lab-hi); }
  .lab-group .lab-item { width: auto; padding: 4px 10px; }
  .lab-group .lab-item[aria-current="true"], .lab-group .lab-item[aria-pressed="true"] { background: rgba(255,255,255,.18); }
  .lab-label { color: var(--lab-dim); }
  .lab-spacer { flex: 1; }
  .lab-stage { display: grid; place-items: center; padding: 32px; overflow: auto; background: var(--stage-bg); }
  .lab-grid { display: grid; gap: 24px 16px; align-items: center; justify-items: start; }
  .lab-cell { cursor: pointer; }
  .lab-cap { font-size: 11px; color: #777; }
</style>
</head>
<body>
<nav class="lab-side" aria-label="Scenarios" id="side"></nav>
<div class="lab-main">
  <header class="lab-bar" id="bar"></header>
  <main class="lab-stage" id="stage"></main>
</div>
<script>
// ---- Per run: replace these ----
const NAME = 'Button';
const VARIANTS = [{ id: 'primary', label: 'Primary' }, { id: 'danger', label: 'Danger' }]; // [] when there are none
const FORCES = ['hover', 'focus', 'pressed']; // [] when there are no pseudo-states
const SCENARIOS = [ // { id, label, group, props }
  { id: 'default', label: 'Default', group: 'Data', props: {} },
  { id: 'loading', label: 'Loading', group: 'Data', props: { loading: true } },
  { id: 'disabled', label: 'Disabled', group: 'Data', props: { disabled: true } },
];
// render(variant, props, force) returns an element: the forked component, with data-force set when force is given
function render(variant, props, force) {
  const el = document.createElement('button');
  el.textContent = props.loading ? 'Saving…' : 'Save';
  el.disabled = !!props.disabled;
  el.dataset.variant = variant;
  if (force) el.dataset.force = force;
  return el;
}
// ---- Harness: leave as is ----
const $ = (s) => document.querySelector(s);
const q = new URLSearchParams(location.search);
const st = { state: q.get('state') || 'component', variant: q.get('variant') || (VARIANTS[0] && VARIANTS[0].id), force: q.get('force') || '' };
const scenario = () => SCENARIOS.find((s) => s.id === st.state);
function sync() { const u = new URL(location); for (const k of ['state', 'variant', 'force']) { st[k] && !(k === 'state' && st[k] === 'component') ? u.searchParams.set(k, st[k]) : u.searchParams.delete(k); } history.replaceState(null, '', u); }
function item(label, on, fn, attr = 'aria-current') { const b = document.createElement('button'); b.className = 'lab-item'; b.textContent = label; if (on) b.setAttribute(attr, 'true'); b.onclick = fn; return b; }
function set(patch) { Object.assign(st, patch); sync(); draw(); }
function drawSide() {
  const side = $('#side'); side.replaceChildren(item('Component', st.state === 'component', () => set({ state: 'component', force: '' })), item('Overview', st.state === 'overview', () => set({ state: 'overview' })));
  side.append(Object.assign(document.createElement('div'), { className: 'lab-rule' }));
  let last;
  for (const s of SCENARIOS) { if (s.group !== last) { const h = document.createElement('h2'); h.textContent = s.group; side.append(h); last = s.group; } side.append(item(s.label, st.state === s.id, () => set({ state: s.id }))); }
}
function drawBar() {
  const bar = $('#bar'); bar.replaceChildren(Object.assign(document.createElement('span'), { className: 'lab-name', textContent: NAME }));
  if (VARIANTS.length > 1) { const g = document.createElement('div'); g.className = 'lab-group'; VARIANTS.forEach((v) => g.append(item(v.label, st.variant === v.id, () => set({ variant: v.id })))); bar.append(g); }
  bar.append(Object.assign(document.createElement('span'), { className: 'lab-spacer' }));
  if (FORCES.length && st.state !== 'component' && st.state !== 'overview') { bar.append(Object.assign(document.createElement('span'), { className: 'lab-label', textContent: 'Force' })); const g = document.createElement('div'); g.className = 'lab-group'; FORCES.forEach((f) => g.append(item(f, st.force === f, () => set({ force: st.force === f ? '' : f }), 'aria-pressed'))); bar.append(g); }
}
function drawStage() {
  const stage = $('#stage'); stage.replaceChildren();
  if (st.state === 'component') return stage.append(render(st.variant, (SCENARIOS[0] || {}).props || {}, ''));
  if (st.state === 'overview') {
    const rows = VARIANTS.length ? VARIANTS : [{ id: undefined, label: '' }];
    const grid = document.createElement('div'); grid.className = 'lab-grid'; grid.style.gridTemplateColumns = `auto repeat(${SCENARIOS.length}, auto)`;
    grid.append(document.createElement('span')); SCENARIOS.forEach((s) => grid.append(Object.assign(document.createElement('span'), { className: 'lab-cap', textContent: s.label })));
    rows.forEach((v) => { grid.append(Object.assign(document.createElement('span'), { className: 'lab-cap', textContent: v.label })); SCENARIOS.forEach((s) => { const c = document.createElement('div'); c.className = 'lab-cell'; c.append(render(v.id, s.props, '')); c.onclick = () => set({ state: s.id, variant: v.id }); grid.append(c); }); });
    return stage.append(grid);
  }
  const s = scenario(); if (s) stage.append(render(st.variant, s.props, st.force));
}
function draw() { drawSide(); drawBar(); drawStage(); }
addEventListener('keydown', (e) => {
  if (e.metaKey || e.ctrlKey || e.altKey || e.shiftKey || (e.target.closest && e.target.closest('input,textarea,select,[contenteditable]'))) return;
  const ids = ['component', 'overview', ...SCENARIOS.map((s) => s.id)]; const i = ids.indexOf(st.state);
  if (e.key === 'ArrowDown') set({ state: ids[Math.min(i + 1, ids.length - 1)] }); else if (e.key === 'ArrowUp') set({ state: ids[Math.max(i - 1, 0)] });
  else if (VARIANTS.length > 1 && (e.key === 'ArrowRight' || e.key === 'ArrowLeft')) { const j = VARIANTS.findIndex((v) => v.id === st.variant) + (e.key === 'ArrowRight' ? 1 : -1); set({ variant: VARIANTS[(j + VARIANTS.length) % VARIANTS.length].id }); }
  else return; e.preventDefault();
});
draw();
</script>
</body>
</html>
```
