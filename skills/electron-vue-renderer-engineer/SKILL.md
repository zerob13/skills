---
name: electron-vue-renderer-engineer
description: Use for every Electron renderer development task that touches Vue, TypeScript, components, styles, routes, composables, Pinia, preload APIs, IPC, UI behavior, or renderer performance. Enforces context-first frontend engineering before code changes.
---

# Electron Vue Renderer Engineer

## Core Rule

Every renderer change begins with engineering context.

Patch only after locating ownership, mapping impact, diagnosing root cause, and selecting the smallest stable implementation.

Frontend changes are application-level changes. A small visual or behavioral change can involve component ownership, DOM placement, state flow, lifecycle, performance, accessibility, and Electron process boundaries.

## Universal Workflow

For every task, follow this loop:

```txt
Locate -> Map -> Diagnose -> Decide -> Patch -> Verify
```

The entry point is universal. Analysis depth is dynamic.

## Required Operating Mode

Before editing code, produce a compact internal working brief using:

```txt
Target
- User-visible behavior:
- Current rendering component:
- Logical owner:
- Route/layout/shell owner:
- Trigger path:
- Existing similar implementation:

Context Map
- Vue owner chain:
- DOM/render chain:
- State source:
- Derived state:
- Events:
- Side effects:
- Styling/layout constraints:
- Performance-sensitive areas:
- Accessibility concerns:
- Electron boundary:
- Existing project patterns:

Diagnosis
- Root cause:
- Correct ownership layer:
- Affected consumers:
- Constraints:
- Existing pattern to reuse:

Decision
- Selected approach:
- Files to edit:
- State impact:
- DOM/layout impact:
- Render/update impact:
- IPC/main-process impact:
- Verification plan:
```

Keep the brief proportional to the task. Use one-line entries for simple changes. Expand dimensions when ownership, state, DOM, async flow, or Electron boundaries are uncertain.

## Analysis Depth

Use the lightest depth that gives a safe decision.

```txt
Depth 1: Local confirmation
- copy, token, icon, class, simple prop, small condition
- clear owner
- stable state source
- no structural DOM/layout change
- no lifecycle/async/IPC effect

Depth 2: Feature-level reasoning
- component behavior change
- props/emits/composable/store involved
- route/layout relationship involved
- async state, watcher, lifecycle, slot, transition, or dynamic component involved

Depth 3: App-level reasoning
- shell/layout/root DOM involved
- overlay, Teleport, global CSS, theme, or z-index involved
- global store, persistence, cache, or cross-route state involved
- preload, IPC, main process, native API, filesystem, or window behavior involved
- list/table/canvas/tree/editor or other performance-sensitive UI involved
```

Start with Depth 1. Escalate when the current context cannot explain the symptom or impact.

## Code Search Baseline

Use targeted search before patching. Expand only as needed.

For Vue ownership and behavior:

```bash
rg -n "targetName|relatedKeyword" src
rg -n "defineProps|defineEmits|computed\\(|watch\\(|watchEffect\\(|onMounted|onUnmounted" src
rg -n "defineStore|storeToRefs|use[A-Z].*Store|use[A-Z].*" src
```

For DOM, layout, and overlay issues:

```bash
rg -n "Teleport|modal|dialog|drawer|popover|tooltip|toast|overlay|portal" src
rg -n "z-index|position: fixed|position: absolute|overflow|transform|filter|perspective|contain|isolation" src
```

For Electron behavior:

```bash
rg -n "contextBridge|ipcRenderer|ipcMain|invoke\\(|send\\(|sendSync|handle\\(|on\\(" .
rg -n "BrowserWindow|webContents|dialog|shell|nativeImage|protocol|Menu|Tray" .
```

For existing conventions:

```bash
rg -n "TODO|FIXME|NOTE|HACK" src electron main preload
find . -maxdepth 3 -type f \( -name "*.vue" -o -name "*.ts" -o -name "*.css" -o -name "*.scss" \) | sort
```

## Dynamic References

Load references by dimension:

- `references/context-first-workflow.md` for all tasks.
- `references/vue-ownership-and-architecture.md` for components, props, slots, composables, routes, layouts, and UI primitives.
- `references/state-reactivity-and-pinia.md` for local state, derived state, watchers, composables, stores, persistence, and store consumers.
- `references/dom-layout-and-layering.md` for CSS, layout, visual hierarchy, overlay, Teleport, stacking context, scrolling, resize, and viewport behavior.
- `references/renderer-performance.md` for DOM count, component count, reactive graph size, list rendering, layout thrash, update paths, and profiling.
- `references/electron-ipc-and-preload.md` for preload APIs, IPC, main process, native APIs, filesystem, windows, menus, and process responsiveness.
- `references/interaction-accessibility.md` for forms, focus, keyboard behavior, dialogs, navigation, ARIA, and reduced motion.
- `references/verification-and-reporting.md` for final checks and response format.
- `references/anti-patterns.md` for common failure modes to actively avoid.

## Engineering Principles

- Reuse existing project patterns first.
- Place state at the narrowest stable owner.
- Place business-domain state in stores.
- Place reusable side effects in composables.
- Place route coordination in route/view components.
- Place layout responsibility in layout or shell components.
- Place visual primitives in shared UI components.
- Place Electron capabilities behind typed preload APIs.
- Keep DOM and reactive graph growth intentional.
- Keep hot paths simple.
- Keep changes minimal and reversible.

## Vue Implementation Rules

- Match the project's Vue style. Prefer Vue 3 Composition API and `<script setup lang="ts">` when the project already uses it.
- Keep computed values pure and deterministic.
- Use watchers for side effects, async synchronization, external systems, and cleanup.
- Keep props stable for frequently updated child components.
- Use `storeToRefs()` when destructuring reactive Pinia state.
- Use `shallowRef()` or `shallowReactive()` for large immutable data or external opaque objects when deep reactivity adds cost.
- Use virtual lists for large visible collections.
- Keep component abstractions out of large hot lists unless reuse and measured value justify the instance cost.
- Prefer typed props, emits, composable returns, and preload APIs.

## DOM / Layout / Overlay Rules

- Map both the Vue logical tree and the actual DOM tree.
- Identify layout constraints from ancestors: overflow, transform, filter, perspective, opacity, contain, isolation, position, and z-index.
- Use Teleport for viewport-level overlays that must escape nested layout constraints.
- Use anchored positioning for popovers, menus, and tooltips tied to a trigger.
- Use centralized tokens or existing scales for z-index, spacing, sizing, and theme values.
- Fix ownership, DOM placement, and layout structure before adding arbitrary offsets or z-index values.
- Keep overlay behavior together: focus, escape key, outside click, scroll lock, aria, stacking, and cleanup.

## Electron Rules

- Keep renderer-facing APIs narrow, typed, and exposed from preload through `contextBridge`.
- Use explicit domain-scoped IPC channel names.
- Use `ipcRenderer.invoke` with `ipcMain.handle` for request/response flows.
- Use `ipcRenderer.send` with `ipcMain.on` for fire-and-forget events.
- Batch, debounce, or throttle high-frequency renderer events before IPC.
- Keep main-process handlers async and responsive.
- Move heavy CPU work to worker threads, utility processes, or dedicated services.
- Clean up IPC listeners during component unmount.
- Return serializable payloads with explicit success/error shapes.

## Verification

Run the smallest meaningful verification set.

```bash
pnpm typecheck
pnpm lint
pnpm test
```

When commands are unavailable, inspect scripts in `package.json` and run the closest equivalent.

For visual/UI changes, verify target behavior, nearby behavior, keyboard behavior, layout under resize, theme variants, and scroll behavior.

For Electron changes, verify renderer responsiveness, preload API surface, IPC listener cleanup, error paths, and main-process latency risk.

## Final Response Format

Report the work using:

```txt
- Changed:
- Root cause:
- Ownership layer:
- State impact:
- DOM/layout impact:
- Performance impact:
- Electron impact:
- Verification:
```
