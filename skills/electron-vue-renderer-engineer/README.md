# Electron Vue Renderer Engineer Skill

A context-first skill for Electron renderer development with Vue.

This skill is designed for tasks where an assistant must modify Vue renderer code safely: components, styles, state, routing, composables, Pinia, preload APIs, IPC, and UI behavior.

The core workflow is universal:

```txt
Locate -> Map -> Diagnose -> Decide -> Patch -> Verify
```

The main idea: every renderer change gets at least a lightweight ownership and impact check. Dialogs, stores, IPC, performance, and accessibility are analysis dimensions, not special entry points.

## Directory

```txt
electron-vue-renderer-engineer/
├─ SKILL.md
├─ README.md
├─ references/
│  ├─ source-notes.md
│  ├─ context-first-workflow.md
│  ├─ vue-ownership-and-architecture.md
│  ├─ state-reactivity-and-pinia.md
│  ├─ dom-layout-and-layering.md
│  ├─ renderer-performance.md
│  ├─ electron-ipc-and-preload.md
│  ├─ interaction-accessibility.md
│  ├─ verification-and-reporting.md
│  └─ anti-patterns.md
├─ templates/
│  ├─ context-map.md
│  ├─ patch-brief.md
│  └─ final-report.md
└─ examples/
   ├─ dialog-layering.md
   ├─ store-derived-state.md
   └─ ipc-renderer-flow.md
```

## Usage

Place this directory under your assistant skills directory, then invoke it for Electron renderer work.

Example instruction:

```txt
Use the electron-vue-renderer-engineer skill. Before editing, build a Context Map and Patch Brief. Patch only after the root cause and ownership layer are clear.
```
