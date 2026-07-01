# Source Notes

Use these sources as background context. Project conventions always win for local implementation details.

## Vue and Pinia

- Vue Teleport guide: https://vuejs.org/guide/built-ins/teleport
  - Teleport solves cases where visual DOM placement needs to escape nested DOM constraints while preserving logical component ownership.
  - Fixed positioning and z-index can be affected by transform, perspective, filter, and containing elements.
- Vue Performance guide: https://vuejs.org/guide/best-practices/performance
  - Separate load performance from update performance.
  - Use props stability, computed stability, `v-once`, `v-memo`, virtualized lists, shallow APIs for large immutable structures, and fewer unnecessary component abstractions in hot paths.
- Vue Watchers guide: https://vuejs.org/guide/essentials/watchers
  - Watchers are for side effects and support cleanup for invalidated async work.
- Vue Accessibility guide: https://vuejs.org/guide/best-practices/accessibility
  - Treat focus, route navigation, semantic structure, contrast, and keyboard behavior as part of UI correctness.
- Pinia docs: https://pinia.vuejs.org/
  - Use stores for global, cross-component, cross-route, or business-domain state.
  - Preserve reactive state with `storeToRefs()` when destructuring.
- Vue SFC script setup docs: https://vuejs.org/api/sfc-script-setup
  - `<script setup>` is the recommended syntax when using Composition API inside SFCs.

## Electron

- Electron IPC guide: https://www.electronjs.org/docs/latest/tutorial/ipc
  - Main and renderer processes communicate through named IPC channels.
  - `ipcRenderer.send` + `ipcMain.on` suits one-way messages.
  - `ipcRenderer.invoke` + `ipcMain.handle` suits request/response flows.
- Electron contextBridge API: https://www.electronjs.org/docs/latest/api/context-bridge
  - Expose narrow renderer APIs from preload when context isolation is enabled.
  - Values crossing the bridge are copied/frozen or proxied depending on type.
- Electron performance guide: https://www.electronjs.org/docs/latest/tutorial/performance
  - Keep CPU, memory, disk, startup, and responsiveness in view.
  - Main process work can affect app responsiveness.

## Skill style inspiration

- antfu/skills: https://github.com/antfu/skills
- vue-best-practices skill: https://github.com/antfu/skills/blob/main/skills/vue-best-practices/SKILL.md
  - Context-first architecture confirmation.
  - Mandatory component boundary planning.
  - Core references loaded early.
  - Optional references loaded by feature dimension.

## Local override rule

When official docs, this skill, and the project disagree, follow this priority:

```txt
user instruction > current project convention > official framework constraints > this skill defaults
```
