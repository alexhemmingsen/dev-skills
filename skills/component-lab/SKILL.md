---
name: component-lab
description: Work on one component in a throwaway lab page that shows it interactive beside every variant and state at once, edit it live with the agent, then lift it into the real component.
disable-model-invocation: true
---

# Component Lab

Invoked with a component ("lets work on the action button"), start at Phase 1. Invoked bare, ask one question: **Which component?**

A lab is a plain white page with a white top bar, split in two. On the left sits the component, interactive, with every action stubbed. On the right is a grid of every state, divided by thin lines, each cell a small label above the real component. You edit the fork, the page updates, and the user sees the change across all states at once. When the user says **lift out**, the work moves into the real component and the lab is deleted.

## Hard Rules

1. **Speed is the product.** Go from the prompt to a drawn lab in the fewest steps. Read the component, write the files, print the URL. Then answer every change request with one edit and nothing else. No browser walkthrough, no screenshots, no console checks, no clicking through scenarios. The user is the one who looks at it.
2. **The lab is throwaway, so this skill's rules replace the project's verification rules for lab files.** Where the project's `CLAUDE.md` or `AGENTS.md` asks for a browser check after every change, a test run, a lint pass, a typecheck or a build, those rules wait until Phase 5. In the lab the loop is: edit, reply to the user, edit, reply.
3. **Fork.** The lab renders a throwaway copy of the component. Production files stay untouched until Phase 5.
4. **Axes come from the code and the prompt.** Make a control for an axis only when the component has it. No variants in the code means no variant tabs.
5. **Every scenario is real.** Product-shaped copy, plausible names and numbers, working interactions. No lorem ipsum.
6. **Everything is in view.** Show all states at once, with no sidebar to click through. Changes ripple across states, so the user watches them together. Axis toggles live in the URL, so a reload keeps the place.
7. **The chrome is fixed.** Copy the top bar, interactive pane and grid from [CHROME.md](CHROME.md). Its look never adapts to the project.
8. **Actions do nothing.** Stub every callback and side effect in the interactive component, so Delete deletes nothing.

## Workflow

### Phase 1: Scope

Resolve the prompt to files. "The buttons" is a **family**: every variant, one row each in the grid. "The action button" is **one variant**: states only. State the scope in one sentence.

**Done when** the component's source files are located and the scope is one sentence.

### Phase 2: Discover axes

Read the component file and the types it imports, nothing wider. Skip the rest of the repo. Find what can change. Two kinds:

- **Variant axes** are what the thing *is*: `variant`, `kind`, `intent`, separate components in a family, `size`.
- **State axes** are what it is *doing*: boolean props (`disabled`, `loading`, `collapsed`), union props (`status`), data branches (empty, one, many, long text, error), context and store values it reads, CSS pseudo-states (`:hover`, `:focus-visible`, `:active`), breakpoints, dark mode.

Use a story or fixture only when the component file leaves the cases unclear.

Group the states by what they are (Data, Interaction, Layout, whatever the component has). Groups set the order of the grid, so related states sit together.

**Done when** every axis and value comes from the code or the user's words.

### Phase 3: Fork and build

Build the lab in the stack's own idiom:

- **Web:** a route `/lab/<component>` (or the framework's equivalent) using the project's tokens, fonts and hot reload. Without a project, one self-contained HTML file.
- **Native or other stacks:** the nearest equivalent that keeps the loop: a debug-only gallery screen with the same layout (interactive component beside a grid of states), a launch argument in place of the URL param, and the stack's own hot reload or preview.

Files: the forked component, one scenarios file (a plain list: `id`, `label`, `group`, `props`, optional `force`), one lab harness built from CHROME.md.

Pseudo-states need a hook. In the fork, every `:hover`, `:focus-visible` and `:active` style gets a twin selector keyed on `data-force`:

```css
.btn:hover, .btn[data-force="hover"] { ... }
```

The harness sets `data-force` on that grid cell's component, so hover shows with no mouse. The interactive component leaves it unset, so real hover works.

**Done when** the files are written. Do not open a browser.

### Phase 4: Talk and change

Print the URL and the discovered list, grouped, in a few lines. Then the user talks and the page changes: "make the danger button rounder", "hover should be darker", "add an empty state". This loop is the whole point, so keep it *instant*:

- **One edit per request.** Edit the fork (or the scenarios file, or the harness) and stop. Every cell and the interactive component render from the same fork, so one edit updates the component in every variant and state at once.
- **Nothing between the edit and the reply.** No re-reading the codebase, no re-running discovery, no browser check, no tests, no lint, no screenshots, whatever the project's own rules say (Hard Rule 2). Say what changed in one line.
- **The page is just HTML.** There is no build step to wait for. With a dev server the hot reload shows the change; with the standalone file the user refreshes.
- **Steering changes the grid too:** "merge those two", "also at mobile width", "drop sizes". Each is one edit to the scenarios file or the harness.

Stay here until the user says **lift out**.

### Phase 5: Lift out

1. Apply the fork's changes to the real component, following the project's conventions.
2. Strip every `data-force` twin selector and any lab-only props.
3. Delete the lab folder, route and scenarios file, unless the user asked to keep them.
4. Run the project's checks. This is the moment its normal verification rules apply, because the work is now production code.

**Done when** the real component shows the result, nothing in production references the lab, no `data-force` remains, and the checks pass.

## Invocation

| Invocation | Behavior |
| --- | --- |
| `<component>` | Phases 1 to 4 |
| `<component> <detail>` | Same, narrowed ("only the destructive variant", "just loading states") |
| `add <scenario>` | New cell in the running lab's grid |
| `lift out` | Phase 5 |
| `lift out, keep the lab` | Phase 5 without deleting the lab |
