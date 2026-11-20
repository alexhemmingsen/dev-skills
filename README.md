# skills

Agent skills for software development.

## Install

```bash
npx skills@latest add alexhemmingsen/skills
```

To update, run `npx skills@latest update`. To pick up new skills, re-run `add`.

## Skills

| Skill | What it does |
| --- | --- |
| `component-lab` | Work on one component in a throwaway lab page: a preview of the interactive component, a grid of every variant and state, and global controls, edit it live, then lift it into the real component. Invoke with `/component-lab <component>`. |
| `prototype-component` | Create a new component: decide which states a component of that type needs, draft a minimal prototype of every one in the same lab page, steer it with the agent, then create the real component. Invoke with `/prototype-component <component>`. Needs `component-lab` installed too. |

## Related libraries

These are installed from their own repos, not from this one:

```bash
npx skills@latest add mattpocock/skills
npx skills@latest add emilkowalski/skills
```

## License

MIT
