---
name: component-lab
description: Work on one component in a throwaway lab page where every variant and state is one click away, edit it live with the agent, then lift it into the real component.
disable-model-invocation: true
---

# Component Lab

Invoked with a component ("lets work on the action button"), start at Phase 1. Invoked bare, ask one question: **Which component?**

A lab is a blank page with a left sidebar. The user clicks a state and sees the component in it, full size. You edit the fork, the page updates, they click the next state. When the user says **lift out**, the work moves into the real component and the lab is deleted.

## Hard Rules

1. **Fork.** The lab renders a throwaway copy of the component. Production files stay untouched until Phase 5.
2. **Axes come from the code and the prompt.** Make a control for an axis only when the component has it. No variants in the code means no variant tabs.
3. **Every scenario is real.** Product-shaped copy, plausible names and numbers, working interactions. No lorem ipsum.
4. **Switching is instant.** No transition between scenarios. The selection lives in the URL, so a reload keeps the place.
5. **The chrome is fixed.** Copy the sidebar and top bar from [CHROME.md](CHROME.md). Its look never adapts to the project.

## Workflow

### Phase 1: Scope

Resolve the prompt to files. "The buttons" is a **family**: every variant, one tab each. "The action button" is **one variant**: states only. State the scope in one sentence.

**Done when** the component's source files are located and the scope is one sentence.

### Phase 2: Discover axes

Read the source and find what can change. Two kinds:

- **Variant axes** are what the thing *is*: `variant`, `kind`, `intent`, separate components in a family, `size`.
- **State axes** are what it is *doing*: boolean props (`disabled`, `loading`, `collapsed`), union props (`status`), data branches (empty, one, many, long text, error), context and store values it reads, CSS pseudo-states (`:hover`, `:focus-visible`, `:active`), breakpoints, dark mode.

Read stories, tests and fixtures too. They name the cases someone already cared about.

Group the states by what they are (Data, Interaction, Layout, whatever the component has).

**Done when** every axis and value is listed with the line of code or the user's words behind it, and nothing is listed without that trace.

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

**Done when** every scenario renders, every interaction responds, the console is clean, and you have clicked through all of them yourself.

### Phase 4: Show and steer

Print the URL and the discovered list, grouped, one line of reason per axis. Then work in the loop: the user says what to change, you edit the fork, the page updates. Steering requests change the list too: "merge those two", "add an empty state", "also at mobile width", "drop sizes". Each is one edit to the scenarios file or the harness.

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
