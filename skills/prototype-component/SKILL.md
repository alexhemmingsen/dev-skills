---
name: prototype-component
description: Create a new component by deciding which states its type needs, prototyping every one in the component lab, and steering it with the agent.
disable-model-invocation: true
---

# Prototype Component

Invoked with a kind of component ("a date picker", "a button"), start at Phase 1. Invoked bare, ask one question: **Which component?**

For a component that **does not exist yet**; `/component-lab` works on an existing one. The result is a **minimal prototype** of every state a component of that type needs, in a throwaway lab page, so the user sees all states at once and steers them with the agent. The user's first question is often "what states does a component like this need?", so the state list is the centerpiece. [lab.html](lab.html) is the page, and its header comment says how it behaves.

## Workflow

### Phase 1: Scope

Search the project once by name. If the component exists, say so in one line and point to `/component-lab`. Stop. Otherwise carry on without asking.

**Done when** nothing by that name exists and the scope is one sentence.

### Phase 2: What states does this need?

**The lab is only as good as its list of states.** Err complete: removing a cell is quicker than noticing a missing one. Two sources decide the list, and the kind of component sets the mix:

- **General rules** define a generic primitive. A button, input, toggle or badge has well-known states (the groups below), so the conventions of its kind carry most of the list.
- **The project** defines a domain component. An order activity card, a customer row or an invoice summary shows the project's own data, so **investigate the project first**: read the model or type behind it (the order: its fields, optional fields, nested lists, relations), every enum and status union (each value is a state), the API shapes and fixtures, the role and permission checks that change what is shown, and the project's rules (`AGENTS.md`, `CLAUDE.md`, design docs, the i18n files). Then read the sibling components of its family for what they already handle. Each status value, each optional field present and absent, each list empty, one and many, and each long or missing text is a cell candidate. The general groups below fill in what the model does not say (loading, error, hover, focus).

Most components mix the two: a domain card is built from primitives with their own states. Name which source each state came from in the printed list.

- **Variants**: the ones people reach for (a button: primary, secondary, destructive, ghost). They become the variant select.
- **Sidebar controls** are the right sidebar. **If it affects all states of the component, it goes in the right sidebar.** That covers theme, size, shape or radius, shadow or elevation, fill color, border, density, text style, width and direction: any prop, class or token that restyles every cell at once. One control per setting, one button per value (three radii are three buttons, not three cells). The test: would a user want to flip it for the whole grid at once? Offer the values a component of this kind ships with (a button: `sm`, `md`, `lg`; `round`, `edgy`), and no more than the project's own scale holds. A theme styles the components only.
- **States** are what it is *doing*, and fill the grid. `loading`, `error` and `disabled` are cells, never controls. In groups:
  - Interaction: default, hover, focus-visible, pressed, disabled; selected or checked for toggles.
  - Async: loading, success, error, where they apply.
  - Data: empty, long content, no text, with an icon, many items, missing optional fields.
  - Layout: narrow, full width, wrapped text.

Take the **look** from the project: its **color variables** for the background dropdown (one search of the CSS custom properties, the Tailwind theme or a tokens file: every variable whose value is a raw color, as `{ name, value }`; aliases and non-colors stay out; an empty list hides the dropdown), one existing component of similar size for radius, type scale and spacing, and its icon source.

Print the list in a few lines: a one-word name per state, each group with a short reason ("Loading: the action takes time"). It answers "what states does it need?", and the user can add and drop states now or in Phase 4.

**Done when** the list covers variants, sidebar controls and every kind of state above that applies, every status value and optional field of the underlying model (for a domain component) is a cell or dropped with a reason, and each state has a one-word name.

### Phase 3: Draft the prototype and open the lab

1. **Draft the component from scratch** in a folder named `ahem-skill-<component>`. Keep it **minimal**: plain and neutral, the project's tokens and fonts, real icons from the project's icon source. The point is that every state exists and reads clearly. Implement every state on the list, and take every sidebar setting as a real prop or class, so each control restyles every state. Give hover, focus and pressed `data-force` twin selectors in the draft's own styles (`.btn:hover, .btn[data-force="hover"] { ... }`).
2. **Make it testable.** Stub every callback and side effect, so the Interactive tab does nothing real. Write scenarios as fixtures of the project's real model type (typed, so a renamed field breaks the lab), with product-shaped copy, real-shaped names, amounts and counts, and plausible values: three rows of "Test item" make every state look fine. Loading and pending states stay pending for good.
3. **Keep it in the cell.** Everything the component renders stays inside its grid cell, panels included. Overlays (listboxes, menus, popovers, tooltips) mount inside the cell: point the portal container at `host`, the fifth argument of `render` and the cell's own element, or render the panel in flow. A panel portaled to `body` escapes the grid.
4. **Fill the page.** Copy [lab.html](lab.html) into the same folder and edit only its `Per run` block: name, variants, axes (the sidebar controls), `COLORS`, scenarios (`id`, `label`, `group`, `props`, optional `force` and `variants`; one per state) and `render(variant, props, force, axes, host)`, which mounts the draft. The rest stays as it is, so the lab looks the same on every run.
5. **Mount it.** Reach the lab with the search param `?ahem-skill=<component>` (kebab-case) on the page the app already serves, so hot reload works. Add one guard where the whole app is rendered (React: `main.tsx` or the root `App`; Next: a small client component rendered from the root layout; the equivalent elsewhere). With the param present it returns the lab instead of the app, so nothing underneath runs. Put it inside the providers the draft needs (theme, i18n). The lab's root is a fixed full-viewport overlay; when it sits in a container with `transform`, `filter`, `backdrop-filter`, `will-change` or `contain`, render it through a portal to `body`. Mark the guard with a comment containing `ahem-skill`: the clean-up search finds it by that marker.
6. **Check every cell, then take the user there.** Start the dev server if it is not running and open `http://localhost:<port>/?ahem-skill=<component>` in the browser. Compare the cell count with the Phase 2 list. Print the URL. With no web dev server (SwiftUI, Android, a plain static site), read [NON-WEB.md](NON-WEB.md): it replaces the mount and the clean-up.

**Done when** the lab is open in the user's browser and has a cell for every state on the Phase 2 list.

### Phase 4: Talk and change

The loop: the user says what to change ("rounder", "add an error state", "what other states are usual?", "drop success"), and each request becomes **one edit** to the draft, the scenarios or the lab page, followed by a one-line reply. A question about states gets an answer and, when the user wants it, a new cell. After a visual change, look at every state again: a fix for one often breaks another.

**Done when** each request ends with its one edit and a one-line reply. The loop ends at `create it` or `drop it`.

### Phase 5: Create the real component and delete the lab

1. Create the component where its kind lives (look at where similar components sit), in the project's conventions for file, name, props, tokens and exports.
2. Remove every `data-force` twin selector.
3. **Delete the lab, always, without waiting to be asked.** Remove the `ahem-skill-<component>` folder and the guard with its import.
4. Prove it is gone: a search of the repo for `ahem-skill` and `data-force` finds nothing, and `git status` lists no lab path. Stop the dev server if you started it.
5. Run the project's checks, now that the work is production code.
6. Tell the user that `/component-lab` (when installed) is how to keep working on it.

**Done when** the component exists in the project, step 4 passes, and the checks pass.

When the user drops the work, run steps 3 and 4 only. While a lab exists, its files stay unstaged and uncommitted: a request to commit, push or open a PR gets a stop until the component is created or the lab deleted.

## Invocation

| Invocation | Behavior |
| --- | --- |
| `<component>` | Phases 1 to 4 |
| `<component> <detail>` | Same, narrowed ("a compact button, no icons") |
| `add <state>` | New cell in the running lab |
| `create it` | Phase 5: create the real component, then delete the lab |
| `drop it` | Phase 5 steps 3 and 4: delete the lab, create nothing |
