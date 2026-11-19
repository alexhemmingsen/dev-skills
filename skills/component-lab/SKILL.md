---
name: component-lab
description: Work on one component in a throwaway lab page that shows it interactive beside every variant and state at once, edit it live with the agent, then lift it into the real component.
disable-model-invocation: true
---

# Component Lab

Invoked with a component ("lets work on the action button"), start at Phase 1. Invoked bare, ask one question: **Which component?**

The lab is a **throwaway** web page, and it always opens in the browser. It sits beside the project: the interactive component first, then a grid of snapshots of every state of the selected variant, with global controls (theme, size, shape) at the side. It adds one marked guard to the project's code and is deleted when the work ends. [lab.html](lab.html) is the page, and its header comment says how it behaves.

## Workflow

### Phase 1: Scope

Search the project once by name. "The buttons" is a **family**: every variant. "The action button" is **one variant**: states only. When nothing matches, or the user says new ("make a new button"), read [NEW-COMPONENT.md](NEW-COMPONENT.md) and follow its notes at each phase.

**Done when** the component's source files are located (or it is new) and the scope is one sentence.

### Phase 2: Discover axes

Read the component file and the types it imports, then find what can change:

- **Variant axes** are what the thing *is* (`variant`, `kind`, `intent`, sibling components of a family). They become the variant select.
- **Global axes** change the look of every instance at once (theme, `size`, shape or rounding, density, width). They become the controls panel. A theme styles the components only.
- **State axes** are what it is *doing*, and fill the grid: boolean and union props, data branches (empty, one, many, long text, error), store and context values, pseudo-states (`:hover`, `:focus-visible`, `:active`), breakpoints.

Group the states (Data, Interaction, Layout, whatever the component has); groups order the grid. A story or fixture helps when the file leaves cases unclear.

Collect the project's **color variables** for the background dropdown with one search (CSS custom properties that hold colors, the Tailwind theme, a tokens file): every variable whose value is a raw color (hex, `rgb()`, `hsl()`, `oklch()`, `color()`) as `{ name, value }`, names and values only. Aliases like `var(--other)` and non-colors stay out. An empty list hides the dropdown.

**Done when** every prop, data branch and pseudo-state in the file is a cell, an axis value, or dropped on purpose, and the color variables are listed.

### Phase 3: Fork and build

1. **Fork.** Copy the component's markup, styles and icon SVGs into a folder named `ahem-skill-<component>`, and trim only what the lab stubs. The fork is the component exactly: markup, its own icons (the SVGs from its icon source), images, fonts, sizes, spacing, colors, radii, shadows. It sits alone on the lab's page background with only its own fill behind it (a button's color, a card's surface, a selected row's highlight), so a nav item from a white sidebar renders without the sidebar's white. An asset you cannot find: say so and ask.
   - **Not a web project** (SwiftUI, Android, anything without web code): there is no markup to copy, so redraw the component in HTML and CSS from its source, reading layout, spacing, fonts, colors, radii and shadows from the view code. Platform icons (SF Symbols and the like) cannot be copied, so draw the closest SVG and tell the user which glyphs are approximations.
2. **Write it fast.** Plain CSS or inline styles with hardcoded values, in whatever form gets it on screen. The project's conventions return at lift-out. A particular width belongs in the fork's own CSS.
3. **Make it testable.** Stub every callback and side effect, so the interactive cell does nothing real (Delete deletes nothing). Give every `:hover`, `:focus-visible` and `:active` style a twin selector keyed on `data-force`, which the lab sets on a grid cell to show the state with no mouse: `.btn:hover, .btn[data-force="hover"] { ... }`. Write scenarios with product-shaped copy and plausible values.
4. **Fill the page.** Copy [lab.html](lab.html) into the same folder and edit only its `Per run` block: name, variants, axes, `COLORS`, scenarios (`id`, `label`, `group`, `props`, optional `force` and `variants`) and `render`, which mounts the fork. The rest stays as it is, so the lab looks the same on every run.
5. **Mount it.**
   - **Web app with a dev server:** reach the lab with the search param `?ahem-skill=<component>` (kebab-case, e.g. `?ahem-skill=text-input`) on the page the app already serves, so hot reload works and the fork imports the component's own modules. Add one guard where the whole app is rendered (React: `main.tsx` or the root `App`; Next: a small client component rendered from the root layout; the equivalent elsewhere). When the param is present it returns the lab instead of the app, so no page, layout, effect, request or redirect underneath runs. Put it inside the providers the component needs (theme, i18n). The lab's root is a fixed full-viewport overlay; if it sits in a container with `transform`, `filter`, `backdrop-filter`, `will-change` or `contain`, render it through a portal to `body`. Mark the guard with a comment containing `ahem-skill` (`// ahem-skill`): it is the only edit to existing code, and the cleanup search finds it by that marker.
   - **No web dev server** (SwiftUI, Android, a plain static site, no project): the lab is the standalone `lab.html`, edited in place. Save it as `ahem-skill-<component>.html` in a temporary directory outside the project, so nothing enters the repo, with the redrawn component in its `render`. The user refreshes the page to see each edit.
6. **Take the user there.** Start the app's dev server if it is not running, open `http://localhost:<port>/?ahem-skill=<component>` in the browser (`open <url>` on macOS, `xdg-open` on Linux) and print the URL. For the standalone file, open its path in the browser. Opening the lab is the hand-off; the user does the looking.

**Done when** the lab is open in the user's browser, served from inside the app.

### Phase 4: Talk and change

Print the axes and states you found in a few lines. Then the loop: the user says what to change ("rounder", "hover darker", "add an empty state", "merge those two"), and each request becomes **one edit** to the fork, the scenarios or the lab page, followed by a one-line reply saying what changed. Every cell renders from the same fork, so one edit updates every variant and state, and hot reload (or a refresh of the standalone file) shows it. The user's eyes are the verification: project rules that ask for a browser check, tests, lint, a typecheck or a build wait for Phase 5.

**Done when** each request ends with its one edit and a one-line reply. The loop ends at `lift out` or `drop it`.

### Phase 5: Lift out and delete the lab

1. Translate the fork's changes into the real component, now in the project's conventions: its tokens, utility classes, naming and file layout. For a non-web project the lab is the design spec: implement what it settled in the project's own language and components.
2. Remove every `data-force` twin selector and any lab-only props from the real component.
3. **Delete the lab, always, without waiting to be asked.** Remove the `ahem-skill-<component>` folder and the guard (the block marked `ahem-skill`) with its import, or, for a standalone file, the `ahem-skill-<component>.html` file.
4. Prove it is gone: a search of the repo for `ahem-skill` and for `data-force` finds nothing, and `git status` lists no lab path. Stop the dev server if you started it.
5. Run the project's checks, now that the work is production code.

**Done when** the real component shows the result (for a non-web project, the SwiftUI or native code implements what the HTML settled), the repo has no trace of the lab (zero hits for `ahem-skill` and `data-force`), and the checks pass.

When the user drops the work ("never mind", "scrap it"), run steps 3 and 4 only.

While a lab exists, its files stay unstaged and uncommitted. A request to commit, push or open a PR gets a stop: say a lab still exists, and lift it out or delete it first.

## Invocation

| Invocation | Behavior |
| --- | --- |
| `<component>` | Phases 1 to 4 |
| `<component> <detail>` | Same, narrowed ("only the destructive variant", "just loading states") |
| `new <component>` | Phases 1 to 4 with [NEW-COMPONENT.md](NEW-COMPONENT.md) |
| `add <scenario>` | New cell in the running lab's grid |
| `lift out` | Phase 5: move the work into the real component, then delete the lab |
| `drop it` | Phase 5 steps 3 and 4: delete the lab without lifting anything out |
