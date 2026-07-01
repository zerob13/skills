# Anti-Patterns

Use this file as a review lens before patching and before final response.

## Symptom-First CSS Patch

Pattern:

```txt
Visible issue -> nearest component -> arbitrary absolute/fixed/z-index/offset patch
```

Better path:

```txt
Visible issue -> DOM/layer map -> stacking/layout diagnosis -> ownership-layer patch
```

Signals:

- Random large `z-index` values.
- `position: absolute` added without a known positioning context.
- Pixel offsets compensate for parent layout bugs.
- Local component CSS tries to solve global overlay behavior.

## Store Creep

Pattern:

```txt
Local UI state -> global store because it is convenient
```

Better path:

```txt
Local state -> parent/feature state -> composable -> store only when shared/persistent/domain-owned
```

Signals:

- Store contains one component's dropdown state.
- Store state mirrors props exactly.
- Components mutate store fields for transient presentation.
- Store reset behavior becomes unclear.

## Watcher Creep

Pattern:

```txt
Computed/derivable value implemented with watch + ref mutation
```

Better path:

```txt
Pure derivation -> computed
Side effect -> watch with explicit source and cleanup
```

Signals:

- Watcher mutates another ref that could be computed.
- Watcher triggers on broad object changes.
- Watcher performs async work without cleanup.
- Multiple watchers encode one state machine implicitly.

## Component Boundary Drift

Pattern:

```txt
Route component grows into data loader + store mutator + layout + primitive renderer + IPC caller
```

Better path:

```txt
Route composes; feature container coordinates; UI components render; composables handle reusable effects; stores hold domain state.
```

Signals:

- Single SFC owns unrelated concerns.
- Child components receive whole stores or huge objects.
- Emit events carry business side effects from leaf components.
- A primitive imports feature stores.

## IPC Chatty Path

Pattern:

```txt
keypress/scroll/drag -> ipcRenderer.invoke/send every event -> main IO/CPU -> renderer waits
```

Better path:

```txt
renderer-local feedback -> batch/debounce/throttle -> async main handler -> explicit loading/result state
```

Signals:

- IPC inside input handlers without debounce.
- Payloads include large objects repeatedly.
- Main handler uses sync filesystem or heavy parsing.
- Renderer blocks interaction while waiting.

## Hidden Lifecycle Leak

Pattern:

```txt
onMounted adds listener/timer/observer/subscription -> component unmounts -> side effect remains
```

Better path:

```txt
create side effect -> register cleanup in onUnmounted/onScopeDispose/watcher cleanup
```

Signals:

- `window.addEventListener` without remove.
- `setInterval` without clear.
- `ResizeObserver` without disconnect.
- `ipcRenderer.on` without unsubscribe.
- Watcher async work races stale state.

## Abstraction Prematurely Added

Pattern:

```txt
First use case -> generic framework/component/composable
```

Better path:

```txt
First use case -> simple local implementation
Second clear use case -> extract stable abstraction
```

Signals:

- Generic names with one caller.
- Many options before requirements exist.
- Abstraction hides ownership rather than clarifying it.
- Large component abstraction inside a hot list.

## Project Convention Drift

Pattern:

```txt
Technically valid Vue/Electron code -> inconsistent with project conventions
```

Better path:

```txt
Search existing patterns -> match naming, files, styles, error shapes, test style, and API boundaries
```

Signals:

- New folders that duplicate existing architecture.
- New z-index scale beside existing tokens.
- New IPC result shape beside existing wrappers.
- Different package manager or command assumptions.
