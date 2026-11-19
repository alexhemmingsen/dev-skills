# Non-web project

Read when the project has no web dev server (SwiftUI, Android, a plain static site, no project). Everything else in [SKILL.md](SKILL.md) applies; these replace the web-app steps.

## Phase 3: redraw and mount

- **Redraw instead of copying.** There is no markup to copy, so draw the component in HTML and CSS from its source, reading layout, spacing, fonts, colors, radii and shadows from the view code. Platform icons (SF Symbols and the like) cannot be copied: draw the closest SVG and tell the user which glyphs are approximations.
- **The lab is the standalone `lab.html`, edited in place.** Save it as `ahem-skill-<component>.html` in a temporary directory outside the project, so nothing enters the repo, with the redrawn component in its `render`. No folder and no guard go into the project.
- **Take the user there** by opening the file's path in the browser. The user refreshes the page to see each edit.

## Phase 5: lift out and delete

The lab is the design spec: implement what it settled in the project's own language and components. Delete the `ahem-skill-<component>.html` file; there is no guard or folder to remove, so the repo search for `data-force` and `ahem-skill` covers the rest.
