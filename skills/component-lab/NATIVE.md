# Native and other stacks

Read when the app has no web dev server (iOS, Android, desktop, other). Everything else in [SKILL.md](SKILL.md) applies; these are the differences.

## Phase 3: mount and hand off

Build the nearest equivalent that keeps the loop: a debug-only gallery screen with the layout described in [lab.html](lab.html)'s header comment (the interactive component first, then a grid of state snapshots, global controls beside it), reached with a launch argument in place of the URL param, on the stack's own hot reload or preview. Mark every edit to existing code with a comment containing `ahem-skill`.

Take the user there: on iOS, build and launch in the simulator straight onto the gallery (`xcodebuild` and `xcrun simctl launch`, or the project's own run command); elsewhere, the stack's own viewer, emulator or preview.

## Phase 5: delete the lab

Remove the gallery screen, the launch-argument handling and any gallery entry. The `ahem-skill` search covers each, since each carries the marker.
