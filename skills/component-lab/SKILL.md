---
name: component-lab
description: Work on one component in a throwaway lab page that shows it interactive beside every variant and state at once, edit it live with the agent, then lift it into the real component.
disable-model-invocation: true
---

# Component Lab

Invoked with a component ("lets work on the action button"), start at Phase 1. Invoked bare, ask one question: **Which component?** When no such component exists yet ("make a new button"), it is a **new component**: the agent drafts it, with every state a finished one needs, and the work goes on from there. The steps marked *New component* below are the only differences.

**The lab is throwaway.** It is a scratch surface beside the project, not part of it. It changes none of the project's code, nothing imports from it, and it is always deleted when the work ends (Hard Rule 11). Treat every lab file as disposable.

A lab is a page (white by default) with a thin top bar and a grid. **Top bar:** a native variant select (only if the component has variants) and a background-color chip that sets the background of the whole page, with the lab's own text, icons and lines switching to a contrast color (components keep theirs). **Grid:** the first cell is the component, interactive, with every action stubbed; every other cell is a non-interactive snapshot of one state of the selected variant, a small label above the real component, divided by thin lines. **Right panel:** global controls that change every instance at once (theme, size, shape, whatever the component has). The theme styles the components only, never the page. You edit the fork, the page updates, and the user sees the change across all states at once. When the user says **lift out**, the work moves into the real component and the lab is deleted.

## Hard Rules

1. **Speed is the product.** Go from the prompt to a drawn lab in the fewest steps. Read the component, write the files, print the URL. Then answer every change request with one edit and nothing else. No browser walkthrough, no screenshots, no console checks, no clicking through scenarios. The user is the one who looks at it.
2. **Throwaway code, so this skill's rules replace the project's verification rules for lab files.** Nothing in the lab ships. Where the project's `CLAUDE.md` or `AGENTS.md` asks for a browser check after every change, a test run, a lint pass, a typecheck or a build, those rules wait until Phase 5. In the lab the loop is: edit, reply to the user, edit, reply.
3. **Fork.** The lab renders a throwaway copy of the component (for a new component, a throwaway first draft). The project's real code stays untouched until Phase 5, and nothing in the project imports from the lab.
4. **Replicate the component exactly** (an existing component; a new one has nothing to replicate, see Phase 3). The fork looks identical to the real component: same markup structure, same icons, same images, same fonts, sizes, spacing, colors, radii and shadows. Icons are the real icons: copy the SVG from the component's own icon source (or import the project's icon module) and render it. An emoji, a text glyph or a placeholder box is never an icon. If the component shows an eye, a chevron or a clear button, the lab shows that exact eye, chevron or clear button. When an asset cannot be found, say so and ask; never invent a stand-in. **Replicate the component, never its surroundings.** The component sits directly on the lab page, so the page background is the only background behind it. Copy the component's own fill (a button's color, a card's surface, a selected row's highlight) and nothing it merely lives inside: no sidebar surface, panel, card, modal, form, page or other container, no wrapper `div` with a color, and no backdrop of any kind. A nav item from a white sidebar is rendered without the sidebar's white; a component with no fill of its own is transparent. The user changes the page color with the background control, and a backdrop would hide it.
5. **Write the fork the quickest way, unrelated to the project.** *How* the fork is written is free: no obligation to the project's CSS conventions, design tokens, utility classes, CSS modules, naming or file layout. Plain CSS or inline styles with hardcoded values, whatever is fastest. The speed is in the writing, never in the likeness. The project's conventions come back at lift-out, when the changes are translated into them.
6. **Axes come from the code and the prompt.** Make a control for an axis only when the component has it. No variants in the code means no variant tabs.
7. **Every scenario is real.** Product-shaped copy, plausible names and numbers, working interactions, the component's real icons. No lorem ipsum.
8. **Everything is in view.** Show all states at once in the grid, with nothing to click through. Changes ripple across states, so the user watches them together. Axis toggles live in the URL, so a reload keeps the place.
9. **The lab looks the same every time.** Copy [lab.html](lab.html) and edit only its `Per run` block. Keep the layout, spacing, colors and behavior as they are, whoever invokes the skill and whatever the project looks like, so the user already knows the page before it loads. Change the look only when the user asks for a different one. [CHROME.md](CHROME.md) describes the page and the Figma design it follows.
10. **Actions do nothing.** Stub every callback and side effect in the interactive component, so Delete deletes nothing.
11. **The lab never survives the work.** It is deleted as the last step of every lab, whether the user lifted it out or dropped it. No lab file is ever left in the app, staged or committed: a leftover route breaks the app's workflow and ships by accident. While a lab exists, never stage or commit lab files, and when the user asks to commit, push or open a PR, stop and say a lab still exists: lift it out or delete it first.

## Workflow

### Phase 1: Scope

Resolve the prompt to files with one search by name. If nothing matches, or the user says new, it is a **new component**: say so in one line ("No button exists yet, so I'll draft one") and go on without asking. "The buttons" is a **family**: every variant, one row each in the grid. "The action button" is **one variant**: states only. State the scope in one sentence.

**Done when** the component's source files are located (or it is a new component) and the scope is one sentence.

### Phase 2: Discover axes

Read the component file and the types it imports, nothing wider. Skip the rest of the repo. Find what can change. Two kinds:

- **Variant axes** are what the thing *is*: `variant`, `kind`, `intent`, separate components in a family. They become the variant select.
- **Global axes** change the look of *all* instances at once: theme, `size`, shape or rounding, density, width. They become the controls panel on the right. A theme axis styles the components only; the page background belongs to the background chip.
- **State axes** are what it is *doing*, and fill the grid: boolean props (`disabled`, `loading`, `collapsed`), union props (`status`), data branches (empty, one, many, long text, error), context and store values it reads, CSS pseudo-states (`:hover`, `:focus-visible`, `:active`), breakpoints, dark mode.

Use a story or fixture only when the component file leaves the cases unclear.

**New component:** there is no code to read, so decide the axes from what a finished component of that kind needs. The variants people reach for, the global axes it should honor (size, shape, theme), and above all **every state**: interaction (default, hover, focus-visible, pressed, disabled), async (loading, success, error where they apply), and data (empty, long content, with an icon, and so on). Err toward complete: the user removes a cell faster than they think of a missing one. For the look, read the color variables below and one existing component of similar size, to take radius, type scale and spacing from the project; skip that when the project has none.

Also collect the project's **color variables** for the background select: one search for CSS custom properties that hold colors (`--color-*`, `--bg-*`, theme blocks), the Tailwind theme or config, or a tokens file. Take **every** variable whose value is a raw color (hex, `rgb()`, `hsl()`, `oklch()`, `color()`), as `{ name, value }` with the raw value, and skip aliases like `var(--other)` and anything that is not a color. Read names and values only, never whole stylesheets. Leave the list empty when the project has none.

Group the states by what they are (Data, Interaction, Layout, whatever the component has). Groups set the order of the grid, so related states sit together.

**Done when** every axis and value comes from the code or the user's words.

### Phase 3: Fork and build

Build the lab in the stack's own idiom:

- **Web app with a dev server:** the lab lives inside the app, served by the app's own dev server, so hot module reload works and the fork can import the component's modules and icons. Read `package.json` and the folder layout to find the router, then pick the mechanism that edits the fewest existing files:
  - **File-based router** (Next `app/` or `pages/`, SvelteKit, Nuxt, Astro, Remix flat routes): add one new route `/ahem-skill-<component>` (component name in kebab-case, e.g. `/ahem-skill-text-input`) the way that router adds any page. No existing file is edited.
  - **Config-based router** (React Router config, TanStack Router, a hand-written routes list, a plain SPA): use a search param, `?ahem-skill=<component>`. Add one guard at the app's render point, inside its providers (theme, i18n, data), that renders the lab page instead of the app when the param is present. The guard is the only edit to existing code, and it is one small block.
  
  Either way, put the fork and the lab page together in one folder named `ahem-skill-<component>` (for example `<routes>/ahem-skill-text-input/`), so deleting the folder deletes the lab, and the `ahem-skill` prefix makes every trace easy to search for. Port [lab.html](lab.html) into it with its CSS and structure unchanged, styled by the fork's own plain CSS. Never write the lab to `/tmp`, a scratchpad or any folder outside the app: a file there gets no hot reload and cannot import anything.
- **No app and no dev server:** one self-contained HTML file, written into the user's working directory.
- **Native or other stacks:** the nearest equivalent that keeps the loop: a debug-only gallery screen with the same layout (interactive component beside a grid of states), a launch argument in place of the URL param, and the stack's own hot reload or preview.

Start the fork by copying the component's real markup, styles and icon SVGs (Hard Rule 4), then strip only what the lab must stub.

**New component:** write a first draft instead, an MVP in the fork from scratch. It implements every scenario in the grid, takes its look from the project (colors, radius, type, spacing), uses real icons from the project's icon source, and stays plain and complete rather than clever. The user then steers it in Phase 4.

Files: the forked component, plus the lab page copied from [lab.html](lab.html) with its `Per run` block filled in (name, variants, axes, `COLORS`, `CELL_WIDTH`, scenarios as a plain list of `id`, `label`, `group`, `props`, optional `force` and `variants`, and a `render` that mounts the fork).

Size the grid to the component. Set `CELL_WIDTH` to a realistic width (e.g. 320) for components that fill their container (inputs, cards, rows) and leave it `0` for components with a natural size (buttons, chips). The grid handles any number of cells, from 1 up. **A component never overflows its cell.** The grid measures what is really rendered, including children and menus that stick out, then widens the columns and heightens the rows to fit, and it scrolls when the window is narrower. So set `CELL_WIDTH` to the width the component needs, not a guess, and leave the grid to do the rest. Fix an overflow by setting `CELL_WIDTH`, or by fixing the component. Never hide it with `overflow: hidden`, never shrink or scale the component to fit, and never edit the lab's grid, cell or slot CSS. Give a scenario `variants` when it only applies to some variants, so each variant's grid holds exactly its own states.

Pseudo-states need a hook. In the fork, every `:hover`, `:focus-visible` and `:active` style gets a twin selector keyed on `data-force`:

```css
.btn:hover, .btn[data-force="hover"] { ... }
```

The harness sets `data-force` on that grid cell's component, so hover shows with no mouse. The interactive component leaves it unset, so real hover works.

Then **take the user there**, in the surface the stack uses:

- **Web app:** start the app's dev server if it is not running, then open the lab (`http://localhost:<port>/ahem-skill-<component>`, or `http://localhost:<port>/?ahem-skill=<component>` for the search-param mechanism) in their browser (`open <url>` on macOS, `xdg-open` on Linux) and print the URL. For the standalone file, open its path.
- **iOS:** build and launch the app in the simulator straight onto the lab screen (the launch argument), e.g. with `xcodebuild` and `xcrun simctl launch`, or the project's own run command.
- **Other stacks:** open the lab in that stack's own viewer, emulator or preview.

Opening the lab is the handoff, not a check. Do not look at it yourself.

**Done when** the files are in the app, served by its dev server, and the lab is open for the user.

### Phase 4: Talk and change

Print the discovered list, grouped, in a few lines. Then the user talks and the page changes: "make the danger button rounder", "hover should be darker", "add an empty state". This loop is the whole point, so keep it *instant*:

- **One edit per request.** Edit the fork (or the scenarios file, or the harness) and stop. Every cell and the interactive component render from the same fork, so one edit updates the component in every variant and state at once.
- **Nothing between the edit and the reply.** No re-reading the codebase, no re-running discovery, no browser check, no tests, no lint, no screenshots, whatever the project's own rules say (Hard Rule 2). Say what changed in one line.
- **The page is just HTML.** There is no build step to wait for. With a dev server the hot reload shows the change; with the standalone file the user refreshes.
- **Steering changes the grid too:** "merge those two", "also at mobile width", "drop sizes". Each is one edit to the scenarios file or the harness.

Stay here until the user says **lift out**.

### Phase 5: Lift out and delete the lab

1. Translate the fork's changes into the real component, now following the project's conventions: its tokens, utility classes, naming and file layout. **New component:** create the real component where its kind lives (look at where similar components sit), following the project's conventions for file, name, props and exports. No existing component is replaced.
2. Strip every `data-force` twin selector and any lab-only props from the real component.
3. **Delete the lab.** Remove the whole `ahem-skill-<component>` folder, and remove everything else the lab touched: the search-param guard, a route registration, a generated route tree or manifest entry, a nav link, a launch argument or gallery entry on native. Deleting is not optional and never waits for the user to ask.
4. Prove it is gone. Search the repo for `ahem-skill` and for `data-force`: zero hits. `git status` lists no lab path. Stop the dev server if you started it.
5. Run the project's checks. This is the moment its normal verification rules apply, because the work is now production code.

**Done when** the real component shows the result, the repo has no trace of the lab (zero hits for `ahem-skill` and `data-force`), and the checks pass.

If the user drops the work instead ("never mind", "scrap it"), skip steps 1 and 2 and run steps 3 and 4.

## Invocation

| Invocation | Behavior |
| --- | --- |
| `<component>` | Phases 1 to 4 |
| `<component> <detail>` | Same, narrowed ("only the destructive variant", "just loading states") |
| `new <component>` | New component: Phase 1 sees nothing to read, so the agent drafts an MVP with every state it needs, then Phase 4 |
| `add <scenario>` | New cell in the running lab's grid |
| `lift out` | Phase 5: move the work into the real component, then delete the lab |
| `drop it` | Delete the lab without lifting anything out |
