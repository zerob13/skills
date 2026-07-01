# Electron IPC and Preload Boundary

Electron renderer work often reaches outside the browser sandbox. Treat preload and IPC as architecture boundaries.

## Process Map

```txt
Vue renderer
  -> preload exposed API
    -> ipcRenderer channel
      -> ipcMain handler
        -> main process service
          -> native API / filesystem / OS / database / network
```

Before changing IPC, map this chain.

```txt
Electron Boundary Map
- Renderer caller:
- Preload API:
- IPC channel:
- Main handler:
- Native/IO dependency:
- Payload shape:
- Error shape:
- Frequency:
- Cancellation/cleanup:
- Responsiveness risk:
```

## Preload API Rules

Expose narrow typed APIs.

```ts
// preload.ts
import { contextBridge, ipcRenderer } from 'electron'

contextBridge.exposeInMainWorld('api', {
  projects: {
    list: () => ipcRenderer.invoke('projects:list'),
    open: (id: string) => ipcRenderer.invoke('projects:open', { id }),
  },
})
```

Renderer declaration:

```ts
// renderer/src/types/electron-api.d.ts
export {}

declare global {
  interface Window {
    api: {
      projects: {
        list: () => Promise<Project[]>
        open: (id: string) => Promise<ProjectOpenResult>
      }
    }
  }
}
```

Guidelines:

- Expose domain methods, not raw Electron modules.
- Keep API names stable and typed.
- Validate payloads at the boundary for untrusted data.
- Keep returned values serializable.
- Keep errors explicit and renderer-safe.

## IPC Pattern Selection

```txt
invoke + handle
- request/response
- async result needed
- native dialog result
- filesystem read/write result
- database query result

send + on
- fire-and-forget
- simple event notification
- no direct result needed

event subscription
- main pushes updates to renderer
- cleanup function required
- typed event payload required

MessagePort / stream / worker
- high-throughput data
- long-running service
- frequent messages
- large payloads
```

## Channel Naming

Use explicit domain names:

```txt
projects:list
projects:open
window:minimize
settings:load
settings:save
files:pick-directory
```

Keep direction and domain clear in code organization.

## Main Handler Rules

```ts
ipcMain.handle('projects:list', async () => {
  const projects = await projectService.list()
  return { ok: true, data: projects }
})
```

Rules:

- Keep handlers async.
- Delegate business logic to services.
- Keep filesystem/database/native work away from UI-critical synchronous paths.
- Validate and normalize inputs.
- Return explicit success/error shapes.
- Avoid sending class instances, functions, DOM objects, or unserializable values.

Example result shape:

```ts
type Result<T, E = string> =
  | { ok: true; data: T }
  | { ok: false; error: E }
```

## Renderer Usage

In Vue, wrap IPC calls in actions or composables.

```ts
export function useProjects() {
  const projects = ref<Project[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function loadProjects() {
    loading.value = true
    error.value = null

    const result = await window.api.projects.list()

    if (result.ok) {
      projects.value = result.data
    }
    else {
      error.value = result.error
    }

    loading.value = false
  }

  return {
    projects,
    loading,
    error,
    loadProjects,
  }
}
```

Guidelines:

- Keep IPC out of templates.
- Centralize calls in stores/composables/services.
- Keep loading/error/cancel state explicit.
- Keep slow operations cancellable or stale-result-safe.

## Event Listener Cleanup

Preload should return unsubscribe functions for subscriptions.

```ts
// preload.ts
contextBridge.exposeInMainWorld('api', {
  windowState: {
    onChanged: (callback: (state: WindowState) => void) => {
      const listener = (_event: Electron.IpcRendererEvent, state: WindowState) => callback(state)
      ipcRenderer.on('window-state:changed', listener)
      return () => ipcRenderer.removeListener('window-state:changed', listener)
    },
  },
})
```

Vue component:

```ts
const stop = window.api.windowState.onChanged((state) => {
  windowState.value = state
})

onUnmounted(stop)
```

## High-Frequency Events

Throttle or batch before crossing IPC:

```txt
- mousemove
- scroll
- resize
- drag
- text input
- progress updates
- file watcher bursts
```

Rules:

- Debounce save-on-input flows.
- Batch filesystem or database updates.
- Coalesce progress events.
- Prefer renderer-local state for immediate interaction feedback.

## Main Process Responsiveness

The main process owns app lifecycle, windows, menus, native dialogs, and coordination. Heavy work there can affect user interaction.

Use:

```txt
- async Node APIs
- worker_threads for CPU-heavy JS
- utilityProcess for isolated services
- child process for external tools
- chunked/batched work for large operations
```

Review risk:

```txt
- Does this handler read/write large files?
- Does it parse huge JSON?
- Does it traverse directories?
- Does it run CPU-heavy transforms?
- Does it wait on synchronous APIs?
- Does renderer await it during typing/drag/scroll?
```

## IPC Review Before Final

```txt
- Renderer API is narrow and typed:
- Channel names are explicit:
- Payloads are serializable:
- Errors are explicit:
- Listener cleanup exists:
- High-frequency calls are batched/throttled:
- Main handler avoids blocking work:
- UI remains responsive on slow path:
```
