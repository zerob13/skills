# Verification and Reporting

Verification should match the change scope. Run the smallest meaningful checks and report what was verified.

## Command Discovery

Inspect package scripts:

```bash
cat package.json
```

Common commands:

```bash
pnpm typecheck
pnpm lint
pnpm test
pnpm test:unit
pnpm build
pnpm dev
```

Use the project's package manager and script names.

## Verification Matrix

```txt
Code correctness
- TypeScript check
- lint
- unit tests
- component tests
- build

Vue behavior
- target behavior
- parent/child communication
- props/emits contract
- store/composable consumers
- lifecycle cleanup

DOM/layout
- actual DOM placement
- stacking context
- scroll container
- resize behavior
- theme variants

Performance
- DOM count
- component count
- watcher/effect count
- list virtualization
- layout read/write cycle
- IPC frequency

Accessibility
- keyboard path
- focus management
- semantic labels
- loading/error/disabled states
- reduced motion

Electron
- preload API shape
- IPC channel and handler
- listener cleanup
- main-process responsiveness
- error path
```

## Visual Verification

For UI changes:

```txt
Visual Verification
- target route/page opened
- target state reached
- expected visual change observed
- nearby UI remains stable
- small viewport checked
- resized window checked
- scroll behavior checked
- theme variants checked when supported
```

For overlay/layout work:

```txt
Overlay Verification
- overlay root inspected
- Teleport target inspected
- stacking order inspected
- focus initial/restore checked
- escape and outside click checked
- scroll lock checked
- nested overlay behavior checked when relevant
```

## Electron Verification

For preload/IPC/main changes:

```txt
Electron Verification
- renderer API is available through window type
- channel reaches expected handler
- success path works
- error path works
- repeated calls behave correctly
- listener cleanup works after unmount/navigation
- slow handler keeps renderer feedback responsive
```

## Final Report Format

Use this format:

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

Keep it factual. Mention commands run and any checks that could not run.

## Example Final Report

```txt
- Changed: Moved the settings dialog to the existing overlay root and reused the shared AppDialog primitive.
- Root cause: The dialog was rendered inside a transformed route panel, which constrained fixed positioning and z-index.
- Ownership layer: Open/close state stays in SettingsView; visual layer belongs to the shared overlay root.
- State impact: No store changes. Local dialog state remains local.
- DOM/layout impact: Dialog DOM now mounts under #overlay-root through Teleport.
- Performance impact: No new watchers. One conditional dialog subtree is mounted only when open.
- Electron impact: No preload, IPC, or main-process changes.
- Verification: Typecheck passed. Manually checked open, close, escape, focus restore, resize, and scroll lock.
```
