# Context-First Workflow

Every renderer change uses the same loop:

```txt
Locate -> Map -> Diagnose -> Decide -> Patch -> Verify
```

The goal is to prevent symptom-level patches. Build enough context to understand ownership and impact, then edit at the correct layer.

## 1. Locate

Identify what currently exists.

```txt
Target
- User-visible behavior:
- Current rendering component:
- Logical owner:
- Route/layout/shell owner:
- Trigger path:
- Existing similar implementation:
```

Questions:

- What exactly changes for the user?
- Which component currently renders it?
- Which component owns the decision?
- Which route, layout, or shell hosts it?
- Which event, prop, store value, watcher, IPC call, or async result drives it?
- Which similar feature already exists in the project?

## 2. Map

Build a proportional context map.

```txt
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
```

Map only what can affect the task. Expand when current knowledge cannot explain the behavior.

## 3. Diagnose

Classify the root cause.

```txt
Diagnosis
- Root cause:
- Correct ownership layer:
- Affected consumers:
- Constraints:
- Existing pattern to reuse:
```

Common diagnosis categories:

```txt
- component ownership mismatch
- DOM placement mismatch
- state ownership mismatch
- derived state bug
- async race
- lifecycle cleanup leak
- CSS containment/layout issue
- accessibility contract issue
- excessive render/update path
- IPC boundary or main-process latency issue
- project convention drift
```

## 4. Decide

Select the smallest stable implementation.

```txt
Decision
- Selected approach:
- Files to edit:
- State impact:
- DOM/layout impact:
- Render/update impact:
- IPC/main-process impact:
- Verification plan:
```

Decision order:

1. Reuse project pattern.
2. Fix the correct ownership layer.
3. Preserve state source stability.
4. Preserve component boundaries.
5. Minimize DOM and reactive graph growth.
6. Add abstraction only when reuse is clear.
7. Touch Electron boundaries only when renderer ownership reaches preload/main.

## 5. Patch

Implement the selected approach.

Patch rules:

- Edit the minimum file set.
- Keep names, imports, styles, tokens, and patterns aligned with nearby code.
- Keep TypeScript types explicit.
- Keep computed values pure.
- Put side effects in watchers, composables, actions, or handlers with cleanup.
- Clean up listeners, timers, observers, intervals, and IPC subscriptions.
- Preserve interaction behavior and accessibility.

## 6. Verify

Verify behavior and engineering impact.

```txt
Verification
- Target behavior works:
- Nearby behavior remains stable:
- State source remains consistent:
- DOM/layout remains correct:
- Re-render risk checked:
- Lifecycle cleanup checked:
- Accessibility checked:
- IPC/main latency checked when applicable:
- Type/lint/test checked when available:
```

## Expansion Triggers

Expand context when any of these appear:

- The visible symptom can be caused by parent layout, global style, route shell, or store state.
- The target is inside a slot, Teleport, Transition, KeepAlive, Suspense, dynamic component, or router view.
- State comes from Pinia, composables, props, persistence, cache, IPC, or async calls.
- CSS depends on overflow, transform, position, z-index, container size, viewport, theme, or platform window chrome.
- Behavior changes during navigation, resizing, theme switch, hydration, async loading, or window focus.
- There are multiple similar implementations in the codebase.

## Small Change Protocol

For a simple local change, use a short version:

```txt
Local Context
- Owner:
- Existing pattern:
- Impact:
- Verification:
```

Even a tiny patch gets this quick ownership check.
