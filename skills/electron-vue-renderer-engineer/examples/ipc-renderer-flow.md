# Example: IPC Renderer Flow Analysis

## Context Map

```txt
Target
- User-visible behavior: Search input freezes while indexing files.
- Current rendering component: FileSearchPanel.vue.
- Logical owner: file search feature composable owns renderer state; main process owns filesystem indexing.
- Route/layout/shell owner: SearchView.vue.
- Trigger path: input event -> search query watcher -> window.api.files.search -> ipc invoke -> main file search service.
- Existing similar implementation: RecentFilesPanel debounces query before IPC.

Context Map
- Vue owner chain: App -> SearchView -> FileSearchPanel.
- DOM/render chain: normal panel.
- State source: local query ref and results ref.
- Derived state: empty/loading/error UI.
- Events: input update.
- Side effects: watcher triggers IPC.
- Styling/layout constraints: none.
- Performance-sensitive areas: typing path.
- Accessibility concerns: loading state announcement.
- Electron boundary: preload files.search -> ipc `files:search` -> main service.
- Existing project patterns: useDebouncedRef and Result<T> IPC shape.
```

## Diagnosis

```txt
Diagnosis
- Root cause: Every keypress triggers IPC and main search work; stale results can overwrite new query results.
- Correct ownership layer: renderer composable controls debounce/stale result handling; main service controls search implementation.
- Affected consumers: FileSearchPanel only.
- Constraints: Preserve existing `files:search` channel and result shape.
- Existing pattern to reuse: RecentFilesPanel debounce pattern.
```

## Decision

```txt
Decision
- Selected approach: Debounce query in renderer, track request id, ignore stale results, keep immediate local input responsive.
- Files to edit: useFileSearch.ts.
- State impact: add local request counter and explicit loading/error.
- DOM/layout impact: none.
- Render/update impact: fewer result updates during typing.
- IPC/main-process impact: lower IPC frequency; no channel shape change.
- Verification plan: type quickly, clear input, slow main response, error path, unmount cleanup.
```
