# Example: Store and Derived State Analysis

## Context Map

```txt
Target
- User-visible behavior: Project list active row updates incorrectly after filtering.
- Current rendering component: ProjectList.vue.
- Logical owner: project selection belongs to project domain store.
- Route/layout/shell owner: ProjectsView.vue.
- Trigger path: search input -> filtered list -> active row rendering.
- Existing similar implementation: UserList uses selected id + computed visible users.

Context Map
- Vue owner chain: App -> ProjectsView -> ProjectList.
- DOM/render chain: normal list inside route view.
- State source: projectStore.projects, projectStore.selectedProjectId, local search query.
- Derived state: filteredProjects and active flags.
- Events: ProjectList emits select(id).
- Side effects: store action may open project through preload API.
- Styling/layout constraints: none.
- Performance-sensitive areas: list can have thousands of rows.
- Accessibility concerns: keyboard selection.
- Electron boundary: openProject action invokes main through preload.
- Existing project patterns: UserList passes stable `active` boolean to rows.
```

## Diagnosis

```txt
Diagnosis
- Root cause: Active row is derived from filtered index instead of stable project id.
- Correct ownership layer: selectedProjectId remains in project store; filtered view state stays local/computed.
- Affected consumers: ProjectList row rendering and keyboard navigation.
- Constraints: Large list path requires stable row props.
- Existing pattern to reuse: UserList active boolean pattern.
```

## Decision

```txt
Decision
- Selected approach: Derive `filteredProjects` with computed, render rows with `active={project.id === selectedProjectId}`, emit id on select.
- Files to edit: ProjectList.vue, ProjectRow.vue only if prop name mismatch.
- State impact: no store shape change.
- DOM/layout impact: none.
- Render/update impact: row prop stability improved; active id change updates only previous and next active rows where possible.
- IPC/main-process impact: no channel change.
- Verification plan: filter, select, keyboard navigate, clear filter, open selected project.
```
