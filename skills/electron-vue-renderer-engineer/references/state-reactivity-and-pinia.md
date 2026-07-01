# State, Reactivity, and Pinia

State changes require ownership clarity. The correct state location is the narrowest stable owner that can explain all consumers and side effects.

## State Ownership Ladder

```txt
Template-only expression
-> component local ref/reactive
-> composable local to a feature
-> parent/feature container state
-> route-level state
-> Pinia store
-> persisted store/cache
-> preload/main-process source
```

Use the lowest level that remains stable.

## State Placement Matrix

```txt
Local component state
- input draft
- open/closed UI state local to one component
- hover/focus/internal interaction state
- temporary validation display state

Parent or feature container state
- multiple child components coordinate the same UI behavior
- feature-level loading/error state
- selected item shared inside one feature
- batch operation state

Composable state
- reusable side effect
- reusable feature logic
- browser API integration
- timer, observer, media query, keyboard, drag/drop, clipboard

Pinia store
- business-domain state
- cross-route or cross-feature state
- persistent/session state
- cache shared by multiple owners
- state synced with IPC/main process

Preload/main source
- filesystem, native dialog, window state, OS integrations
- privileged API results
- long-lived desktop app services
```

## Source of Truth

Before editing state, answer:

```txt
- What is the authoritative source?
- Which values are derived?
- Which consumers read it?
- Which actions mutate it?
- Which side effects observe it?
- Is it persisted or restored?
- Does it cross renderer/preload/main?
```

One source of truth keeps stores stable and components predictable.

## Computed vs Watch

Use `computed` for pure derived values.

```ts
const visibleItems = computed(() =>
  items.value.filter(item => item.enabled && matchesQuery(item, query.value)),
)
```

Use `watch` or `watchEffect` for side effects.

```ts
watch(
  () => props.userId,
  async (userId, _previousUserId, onCleanup) => {
    const controller = new AbortController()
    onCleanup(() => controller.abort())

    profile.value = await fetchProfile(userId, { signal: controller.signal })
  },
)
```

Guidelines:

- Keep computed getters pure.
- Keep watchers side-effect-focused.
- Add cleanup for async work, listeners, timers, observers, and subscriptions.
- Prefer explicit `watch` sources when rerun conditions should be obvious.
- Use `watchEffect` when automatic dependency tracking is clearer and the effect remains small.

## Pinia Store Rules

Pinia stores are domain state and business operations.

```ts
export const useProjectStore = defineStore('project', () => {
  const projects = ref<Project[]>([])
  const selectedProjectId = ref<string | null>(null)

  const selectedProject = computed(() =>
    projects.value.find(project => project.id === selectedProjectId.value) ?? null,
  )

  async function loadProjects() {
    projects.value = await window.api.projects.list()
  }

  return {
    projects,
    selectedProjectId,
    selectedProject,
    loadProjects,
  }
})
```

Rules:

- State is source data.
- Getters/computed values derive from source data.
- Actions perform domain operations and side effects.
- Components call actions and render state.
- Cross-store dependencies stay explicit and acyclic.
- Store names and action names match domain language.

## `storeToRefs()`

When destructuring a store in a component, preserve reactivity:

```ts
const projectStore = useProjectStore()
const { projects, selectedProject } = storeToRefs(projectStore)
const { loadProjects } = projectStore
```

Use direct method destructuring for actions. Use `storeToRefs()` for state/getters.

## Local UI State vs Store State

Keep local ephemeral UI state local:

```ts
const isMenuOpen = ref(false)
const draftTitle = ref('')
```

Promote to store only when:

```txt
- another route or feature needs it
- persistence/restore needs it
- a domain operation owns it
- preload/main synchronization needs it
- multiple non-parent-related consumers coordinate through it
```

## Async State

Use explicit status fields or discriminated unions.

```ts
type LoadState<T> =
  | { status: 'idle'; data: null; error: null }
  | { status: 'loading'; data: T | null; error: null }
  | { status: 'success'; data: T; error: null }
  | { status: 'error'; data: T | null; error: Error }
```

Avoid hidden async state encoded only by `null` values when UI has multiple states.

## Large or External Data

For large immutable structures or external opaque objects:

```ts
const rows = shallowRef<Row[]>([])
const editorInstance = shallowRef<Editor | null>(null)
```

Guidelines:

- Use deep reactivity for ordinary UI state.
- Use shallow APIs for large immutable arrays/objects or external class instances.
- Replace shallow values immutably to trigger updates.
- Mark external non-reactive objects with `markRaw()` when appropriate.

## State Change Review

Before finishing a state patch:

```txt
- Source of truth remains single:
- Derived state remains computed:
- Watchers have cleanup:
- Store consumers remain stable:
- Persistence shape remains compatible:
- IPC payload shape remains compatible:
- Reset/logout/navigation behavior remains correct:
```
