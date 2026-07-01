# Example: Dialog Layering Analysis

## Context Map

```txt
Target
- User-visible behavior: Settings dialog appears behind the sidebar and is clipped by the content panel.
- Current rendering component: SettingsDialog.vue.
- Logical owner: SettingsView.vue owns open/close state.
- Route/layout/shell owner: AppShell.vue owns sidebar/content layout and overlay root.
- Trigger path: SettingsView button -> isSettingsOpen -> SettingsDialog.
- Existing similar implementation: AppDialog + #overlay-root used by ConfirmDeleteDialog.

Context Map
- Vue owner chain: App -> AppShell -> RouterView -> SettingsView -> SettingsDialog.
- DOM/render chain: #app -> .app-shell -> .content-panel -> SettingsDialog.
- State source: local ref in SettingsView.
- Derived state: none.
- Events: close emitted by dialog.
- Side effects: escape key and scroll lock handled by AppDialog in existing pattern.
- Styling/layout constraints: .content-panel has overflow hidden; .app-shell has transform for animation.
- Performance-sensitive areas: dialog only mounted when open.
- Accessibility concerns: focus trap and restore required.
- Electron boundary: none.
- Existing project patterns: ConfirmDeleteDialog uses Teleport to #overlay-root and AppDialog primitive.
```

## Diagnosis

```txt
Diagnosis
- Root cause: Dialog DOM is inside transformed/clipping layout ancestors.
- Correct ownership layer: open/close state remains in SettingsView; visual layer belongs to #overlay-root via AppDialog.
- Affected consumers: SettingsView only.
- Constraints: Reuse existing AppDialog behavior and z-index tokens.
- Existing pattern to reuse: ConfirmDeleteDialog.
```

## Decision

```txt
Decision
- Selected approach: Wrap SettingsDialog surface in the existing AppDialog and Teleport to #overlay-root.
- Files to edit: SettingsView.vue and SettingsDialog.vue only if API alignment is needed.
- State impact: local state unchanged.
- DOM/layout impact: rendered DOM moves to overlay root; logical owner unchanged.
- Render/update impact: no new watcher; conditional subtree remains mounted only when open.
- IPC/main-process impact: none.
- Verification plan: open/close, escape, outside click, focus restore, scroll lock, resize, sidebar overlap.
```

## ASCII UI

```txt
Before
body
└─ #app
   └─ .app-shell [transform]
      ├─ .sidebar [z: 20]
      └─ .content-panel [overflow: hidden]
         └─ .settings-dialog [z: 999, clipped]

After
body
├─ #app
│  └─ .app-shell
│     ├─ .sidebar
│     └─ .content-panel
└─ #overlay-root
   └─ .app-dialog-layer
      └─ .settings-dialog
```
