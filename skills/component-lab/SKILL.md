---
name: component-lab
description: Work on one component in a throwaway lab page where every variant and state is one click away, edit it live with the agent, then lift it into the real component.
disable-model-invocation: true
---

# Component Lab

Invoked with a component ("lets work on the action button"), start at Phase 1. Invoked bare, ask one question: **Which component?**

A lab is a blank page with a left sidebar. The user clicks a state and sees the component in it, full size. You edit the fork, the page updates, they click the next state. When the user says **lift out**, the work moves into the real component and the lab is deleted.

## Hard Rules

1. **Speed is the product.** Go from the prompt to a drawn lab in the fewest steps. Read the component, write the files, print the URL. No browser walkthrough, no screenshots, no console checks, no clicking through scenarios. The user is the one who looks at it.
2. **Fork.** The lab renders a throwaway copy of the component. Production files stay untouched until Phase 5.
3. **Axes come from the code and the prompt.** Make a control for an axis only when the component has it. No variants in the code means no variant tabs.
4. **Every scenario is real.** Product-shaped copy, plausible names and numbers, working interactions. No lorem ipsum.
5. **Switching is instant.** No transition between scenarios. The selection lives in the URL, so a reload keeps the place.
6. **The chrome is fixed.** Copy the sidebar and top bar from [CHROME.md](CHROME.md). Its look never adapts to the project.

## Workflow

### Phase 1: Scope

Resolve the prompt to files. "The buttons" is a **family**: every variant, one tab each. "The action button" is **one variant**: states only. State the scope in one sentence.

**Done when** the component's source files are located and the scope is one sentence.

### Phase 2: Discover axes

Read the component file and the types it imports, nothing wider. Skip the rest of the repo. Find what can change. Two kinds:

- **Variant axes** are what the thing *is*: `variant`, `kind`, `intent`, separate components in a family, `size`.
- **State axes** are what it is *doing*: boolean props (`disabled`, `loading`, `collapsed`), union props (`status`), data branches (empty, one, many, long text, error), context and store values it reads, CSS pseudo-states (`:hover`, `:focus-visible`, `:active`), breakpoints, dark mode.

Use a story or fixture only when the component file leaves the cases unclear.

Group the states by what they are (Data, Interaction, Layout, whatever the component has).

**Done when** every axis and value comes from the code or the user's words.

### Phase 3: Fork and build

Build the lab in the stack's own idiom:

- **Web:** a route `/lab/<component>` (or the framework's equivalent) using the project's tokens, fonts and hot reload. Without a project, one self-contained HTML file.
- **Native or other stacks:** the nearest equivalent that keeps the loop: a debug-only gallery screen with the same sidebar structure, a launch argument in place of the URL param, and the stack's own hot reload or preview.

Files: the forked component, one scenarios file (a plain list: `id`, `label`, `group`, `props`), one lab harness built from CHROME.md.

Pseudo-states need a hook. In the fork, every `:hover`, `:focus-visible` and `:active` style gets a twin selector keyed on `data-force`:

```css
.btn:hover, .btn[data-force="hover"] { ... }
```

The harness sets `data-force` on the component, so hover shows with no mouse. The Component tab leaves it unset, so real hover works.

**Done when** the files are written. Do not open a browser.

### Phase 4: Show and steer

Print the URL and the discovered list, grouped, in a few lines. Then work in the loop: the user says what to change, you edit the fork, the page updates. Steering requests change the list too: "merge those two", "add an empty state", "also at mobile width", "drop sizes". Each is one edit to the scenarios file or the harness.

Stay here until the user says **lift out**.

### Phase 5: Lift out

1. Apply the fork's changes to the real component, following the project's conventions.
2. Strip every `data-force` twin selector and any lab-only props.
3. Delete the lab folder, route and scenarios file, unless the user asked to keep them.
4. Run the project's checks.

**Done when** the real component shows the result, nothing in production references the lab, no `data-force` remains, and the checks pass.

## Invocation

| Invocation | Behavior |
| --- | --- |
| `<component>` | Phases 1 to 4 |
| `<component> <detail>` | Same, narrowed ("only the destructive variant", "just loading states") |
| `add <scenario>` | New sidebar item in the running lab |
| `lift out` | Phase 5 |
| `lift out, keep the lab` | Phase 5 without deleting the lab |
