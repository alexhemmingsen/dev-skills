# New component

Read when a name search finds nothing, or the user says new ("make a new button"). Say so in one line ("No button exists yet, so I'll draft one") and carry on without asking. Everything else in [SKILL.md](SKILL.md) applies; these are the differences.

## Phase 2: decide the axes

There is no code to read, so decide the axes from what a finished component of that kind needs:

- the variants people reach for, and the global axes it should honor (size, shape, theme);
- **every state**: interaction (default, hover, focus-visible, pressed, disabled), async (loading, success, error where they apply), and data (empty, long content, with an icon).

Err toward complete: removing a cell is quicker than noticing a missing one. Take the look from the project: the color variables collected in Phase 2 and one existing component of similar size, for radius, type scale and spacing.

**Done when** the list covers variants, global axes and all three kinds of state, and every state has a one-word name.

## Phase 3: draft an MVP

Write the first draft in the fork from scratch instead of copying: every scenario implemented, the project's look, real icons from the project's icon source, plain and complete. The user steers it in Phase 4.

## Phase 5: create the real component

Create it where its kind lives (look at where similar components sit), in the project's conventions for file, name, props and exports. No existing component is replaced. Steps 2 to 5 follow as usual.
