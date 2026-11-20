---
name: component-lab
description: Work on one component in a throwaway lab page that shows it interactive beside every variant and state at once, edit it live with the agent, then lift it into the real component.
disable-model-invocation: true
---

# Component Lab

Invoked with a component ("lets work on the action button"), start at Phase 1. Invoked bare, ask one question: **Which component?**

The lab is a **throwaway** web page, and it always opens in the browser. It sits beside the project: the interactive component first, then a grid of snapshots of every state of the selected variant, with global controls (theme, size, shape) at the side. It renders the project's **real component**, imported untouched, so styling is the production styling. It adds one marked guard to the project's code and is deleted when the work ends. [lab.html](lab.html) is the page, and its header comment says how it behaves.

## Workflow

### Phase 1: Scope

Search the project once by name. "The buttons" is a **family**: every variant. "The action button" is **one variant**: states only. When nothing matches, or the user says new ("make a new button"), read [NEW-COMPONENT.md](NEW-COMPONENT.md) and follow its notes at each phase.

**Done when** the component's source files are located (or it is new) and the scope is one sentence.

### Phase 2: Discover axes

**The lab is only as good as its list of states.** A missing state is a bug nobody sees until production, so investigate until you are sure the list is complete. Do not stop at the props.

Read the component file, every file it imports that shapes its output (child components, style files, variant helpers such as `cva` or a `variants` map, hooks, stores, contexts, i18n keys), and the types it uses. Then find what can change:

- **Variant axes** are what the thing *is* (`variant`, `kind`, `intent`, sibling components of a family). They become the variant select.
- **Global axes** change the look of every instance at once (theme, `size`, shape or rounding, density, width). They become the controls panel. A theme styles the components only.
- **State axes** are what it is *doing*, and fill the grid.

**Find the states by reading, not by guessing.** Walk the component top to bottom and write down every branch:

- every prop, every value of every union and enum, both values of every boolean, and the combinations the code branches on (`disabled` and `loading` together, `selected` and `error`);
- every conditional in the render (`&&`, ternaries, early returns, `switch`), and every conditional class or style;
- data branches: empty, one, many, long text, no text, missing optional fields, error;
- store, context, query and feature-flag values it reads, and the role or plan checks;
- pseudo-states in the styles (`:hover`, `:focus-visible`, `:active`, `:disabled`, `[aria-*]`, `[data-*]`), and breakpoints or container queries;
- states shown in stories, tests or fixtures that the file leaves unclear.

A state the code cannot reach is not a state. List it as missing and leave it out.

Group the states (Data, Interaction, Layout, whatever the component has); groups order the grid. Write the full list down, one line each, before building.

Collect the project's **color variables** for the background dropdown with one search (CSS custom properties that hold colors, the Tailwind theme, a tokens file): every variable whose value is a raw color (hex, `rgb()`, `hsl()`, `oklch()`, `color()`) as `{ name, value }`, names and values only. Aliases like `var(--other)` and non-colors stay out. An empty list hides the dropdown.

**Done when** every prop value, branch and pseudo-state in the code is a cell, an axis value, or dropped on purpose with a reason, and the color variables are listed. Say how many variants and states you found.

### Phase 3: Render the real component and build

With no web dev server (SwiftUI, Android, a plain static site, no project), read [NON-WEB.md](NON-WEB.md): it replaces step 1, step 7 and the clean-up in Phase 5.

1. **Import the real component.** Put the lab in a folder named `ahem-skill-<component>` and import the component from the project, **untouched**. It brings its own markup, icons, fonts, sizes, spacing, colors, radii and shadows, and the app's global styles, providers and theme. Do not copy or rewrite its markup or styles: a hand copy drifts from production, and the styling is where it drifts first. The lab adds only the page, the scenarios and the controls, with no fonts, styles or themes of its own on the component. Where the component needs the surroundings it has in production (a parent's text color, a width, a provider), supply those the way production does.
   Fork a copy into the folder only when the real component cannot be rendered in place: a new component (see [NEW-COMPONENT.md](NEW-COMPONENT.md)), or one that cannot be mounted without editing it. Say so in one line, and ask before editing the real component.
2. **Feed each state at the data boundary.** Drive the real component from the lab without editing it, in this order: props, then the app's own data seam (a seeded query cache, a mock provider, the project's request mocks, a store set before render). A fixture threaded through five levels of internals tests a path production never takes. Where the only way in is editing the component, say so and ask before adding a seam.
3. **Keep it in the cell.** Everything a component renders stays inside its grid cell, panels included. Overlays (listboxes, menus, popovers, date pickers, tooltips) mount inside the cell: point the component's portal container at `host`, the fifth argument of `render` and the cell's own element, or render the panel in flow. A panel portaled to `body` escapes the grid. A panel opened in the interactive cell may overlap the cells around it, since it closes again; the lab keeps the grid still.
4. **Make it testable.** Stub every callback and side effect, so the interactive cell does nothing real (Delete deletes nothing). Write scenarios with product-shaped copy, real-shaped names, amounts and counts, and plausible values; three rows of "Test item" make every state look fine. Loading and pending states stay pending, with no timeout that resolves them.
5. **Force the pseudo-states.** `:hover`, `:focus-visible` and `:active` need a twin selector keyed on `data-force`, which the lab sets on a grid cell to show the state with no mouse: `.btn:hover, .btn[data-force="hover"] { ... }`. Add the twins to the component's own styles as a temporary, marked edit (comment `ahem-skill`), tell the user you did, and remove them in Phase 5. Where the hover styles are utility classes that cannot take a twin, put the same declarations in a lab rule keyed on `data-force` and name those cells as approximations.
6. **Fill the page.** Copy [lab.html](lab.html) into the same folder and edit only its `Per run` block: name, variants, axes, `COLORS`, scenarios (`id`, `label`, `group`, `props`, optional `force` and `variants`) and `render(variant, props, force, axes, host)`, which mounts the real component. The rest stays as it is, so the lab looks the same on every run.
7. **Mount it.** Reach the lab with the search param `?ahem-skill=<component>` (kebab-case, e.g. `?ahem-skill=text-input`) on the page the app already serves, so hot reload works. Add one guard where the whole app is rendered (React: `main.tsx` or the root `App`; Next: a small client component rendered from the root layout, which must be client code so fixtures cross no server boundary; the equivalent elsewhere). When the param is present it returns the lab instead of the app, so no page, layout, effect, request or redirect underneath runs. Put it inside the providers the component needs (theme, i18n, query client). The lab's root is a fixed full-viewport overlay; if it sits in a container with `transform`, `filter`, `backdrop-filter`, `will-change` or `contain`, render it through a portal to `body`. Mark the guard with a comment containing `ahem-skill` (`// ahem-skill`): the cleanup search finds it by that marker.
8. **Check every cell, then take the user there.** Start the app's dev server if it is not running and open `http://localhost:<port>/?ahem-skill=<component>` in the browser (`open <url>` on macOS, `xdg-open` on Linux). Compare the lab's cell count with the state list from Phase 2, and check that each cell shows the mock data, not the real data and not a blank. A fixture that never appears (a refetch over the seed, live sync, a server boundary) is the common failure: fix the plumbing before handing over. With no browser to inspect, say so. Print the URL.

**Done when** the lab is open in the user's browser, and it has a cell for every state on the Phase 2 list.

### Phase 4: Talk and change

Print the axes and states you found in a few lines. Then the loop: the user says what to change ("rounder", "hover darker", "add an empty state", "merge those two"), and each request becomes **one edit** to the component, the scenarios or the lab page, followed by a one-line reply saying what changed. Edits to the component go straight into the real file, in the project's conventions (its tokens and classes), unless it is a fork. Every cell renders the same component, so one edit updates every variant and state, and hot reload (or a refresh of the standalone file) shows it. After a visual change, look at every state again: a fix for one often breaks another. The user's eyes are the verification: project rules that ask for a browser check, tests, lint, a typecheck or a build wait for Phase 5.

**Done when** each request ends with its one edit and a one-line reply. The loop ends at `lift out` or `drop it`.

### Phase 5: Lift out and delete the lab

1. With a fork, translate its changes into the real component, in the project's conventions: its tokens, utility classes, naming and file layout. With the real component rendered in place, its edits are already there.
2. Remove every `data-force` twin selector and any lab-only props from the real component.
3. **Delete the lab, always, without waiting to be asked.** Remove the `ahem-skill-<component>` folder and the guard (the block marked `ahem-skill`) with its import.
4. Prove it is gone: a search of the repo for `ahem-skill` and for `data-force` finds nothing, and `git status` lists no lab path. Stop the dev server if you started it.
5. Run the project's checks, now that the work is production code.

**Done when** the real component shows the result, the repo has no trace of the lab (zero hits for `ahem-skill` and `data-force`), and the checks pass.

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
