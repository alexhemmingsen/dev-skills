---
name: prototype-component
description: Create a new component: decide which states its type needs, prototype every one in the component lab, steer it with the agent, then create the real component.
disable-model-invocation: true
---

# Prototype Component

Invoked with a kind of component ("a date picker", "a button"), start at Phase 1. Invoked bare, ask one question: **Which component?**

For a component that **does not exist yet**; `/component-lab` works on an existing one. The result is a **minimal prototype** of every state a component of that type needs, in the same throwaway page `/component-lab` opens ([lab.html](../component-lab/lab.html)), so the user sees all states at once and steers them with the agent. The user's first question is often "what states does a component like this need?", so the state list is the centerpiece. This skill reads [component-lab's SKILL.md](../component-lab/SKILL.md) for the lab mechanics, so it needs `component-lab` installed beside it.

## Workflow

### Phase 1: Scope

Search the project once by name. If the component exists, say so in one line and point to `/component-lab`. Stop. Otherwise carry on without asking.

**Done when** nothing by that name exists and the scope is one sentence.

### Phase 2: What states does this need?

**The lab is only as good as its list of states.** Decide it from what a finished component of this kind needs, and err complete: removing a cell is quicker than noticing a missing one.

- **Variants**: the ones people reach for (a button: primary, secondary, destructive, ghost). They become the variant select.
- **Sidebar controls**: if it affects all states, it goes in the right sidebar (component-lab Phase 2 says how). Size, shape or radius, shadow, theme, density, width, fill: one control per setting, one button per value.
- **States**, in groups:
  - Interaction: default, hover, focus-visible, pressed, disabled; selected or checked for toggles.
  - Async: loading, success, error, where they apply.
  - Data: empty, long content, no text, with an icon, many items, missing optional fields.
  - Layout: narrow, full width, wrapped text.

Take the **look** from the project: its color variables (component-lab Phase 2 says how to collect them), one existing component of similar size for radius, type scale and spacing, and its icon source.

Print the list in a few lines: a one-word name per state, each group with a short reason ("Loading: the action takes time"). It answers "what states does it need?", and the user can add and drop states now or in Phase 4.

**Done when** the list covers variants, sidebar controls and every kind of state above that applies, and each state has a one-word name.

### Phase 3: Draft the prototype and open the lab

1. **Draft the component from scratch** in a folder named `ahem-skill-<component>`. Keep it **minimal**: plain and neutral, the project's tokens and fonts, real icons from the project's icon source. The point is that every state exists and reads clearly. Implement every state on the list, with hover, focus and pressed as `data-force` twin selectors in the draft's own styles (`.btn:hover, .btn[data-force="hover"] { ... }`).
2. **Build the lab** as component-lab's Phase 3 says, from step 2 on. The draft is already in the folder, so it renders as it is. One scenario per state, one sidebar control per setting from Phase 2, the same guard marked `ahem-skill`. With no web dev server, component-lab's [NON-WEB.md](../component-lab/NON-WEB.md) applies.
3. **Open it in the browser** and print the URL.

**Done when** the lab is open in the user's browser and has a cell for every state on the Phase 2 list.

### Phase 4: Talk and change

Print the variants, sidebar controls and states in a few lines. Then the loop: the user says what to change ("rounder", "add an error state", "what other states are usual?", "drop success"), and each request becomes **one edit** to the draft, the scenarios or the lab page, followed by a one-line reply. A question about states gets an answer and, when the user wants it, a new cell. After a visual change, look at every state again: a fix for one often breaks another.

**Done when** each request ends with its one edit and a one-line reply. The loop ends at `create it` or `drop it`.

### Phase 5: Create the real component and delete the lab

1. Create the component where its kind lives (look at where similar components sit), in the project's conventions for file, name, props, tokens and exports. It sits beside the existing components and replaces none.
2. Remove every `data-force` twin selector.
3. **Delete the lab, always, without waiting to be asked.** Remove the `ahem-skill-<component>` folder and the guard with its import.
4. Prove it is gone: a search of the repo for `ahem-skill` and `data-force` finds nothing, and `git status` lists no lab path. Stop the dev server if you started it.
5. Run the project's checks, now that the work is production code.
6. Tell the user `/component-lab` is how to keep working on it.

**Done when** the component exists in the project, the repo has zero hits for `ahem-skill` and `data-force`, and the checks pass.

When the user drops the work, run steps 3 and 4 only. While a lab exists, its files stay unstaged and uncommitted: a request to commit, push or open a PR gets a stop until the component is created or the lab deleted.

## Invocation

| Invocation | Behavior |
| --- | --- |
| `<component>` | Phases 1 to 4 |
| `<component> <detail>` | Same, narrowed ("a compact button, no icons") |
| `add <state>` | New cell in the running lab |
| `create it` | Phase 5: create the real component, then delete the lab |
| `drop it` | Phase 5 steps 3 and 4: delete the lab, create nothing |
