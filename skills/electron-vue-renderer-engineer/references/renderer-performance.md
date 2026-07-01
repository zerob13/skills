# Renderer Performance

Renderer performance includes load performance, update performance, DOM size, reactive graph size, layout cost, and Electron responsiveness.

## Performance Map

Before performance-sensitive patches, identify:

```txt
Performance Map
- User interaction path:
- Components updated:
- DOM nodes added/removed:
- Reactive sources changed:
- Computed values touched:
- Watchers/effects added:
- Lists/tables/tree/editor/canvas affected:
- Layout measurement/mutation pattern:
- IPC calls during interaction:
- Existing profiling evidence:
```

## Update Path

For a given user action:

```txt
Event -> reactive write -> computed invalidation -> component update -> DOM patch -> layout/style/paint -> IPC/native side effect
```

Find the widest fan-out point and shrink it.

## Props Stability

Keep child props stable in frequently updated parents.

Less stable:

```vue
<ListItem
  v-for="item in items"
  :key="item.id"
  :item="item"
  :active-id="activeId"
/>
```

More stable:

```vue
<ListItem
  v-for="item in items"
  :key="item.id"
  :item="item"
  :active="item.id === activeId"
/>
```

Only the affected children receive prop changes.

## Computed Stability

- Keep computed values pure.
- Return stable primitives where possible.
- Avoid creating new object/array instances in hot computed paths unless needed.
- Memoize expensive derived values when dependencies are clear.
- Move heavy transformations away from templates.

## Watcher Cost

Watchers are side effects. Review:

```txt
- What reactive sources trigger it?
- How often does it run?
- Does it perform IO, IPC, DOM measurement, or expensive transformation?
- Does it clean up invalid async work?
- Does it run in a hot typing/drag/scroll path?
```

High-frequency watchers should debounce, throttle, batch, or move work away from the interaction path.

## Large Lists

For large lists, tables, trees, or logs:

- Use virtualization when visible DOM count is large.
- Keep row components lightweight.
- Keep row props stable.
- Avoid injecting heavy providers per row.
- Avoid watchers per row unless necessary.
- Avoid measuring every row synchronously during update.
- Use pagination, windowing, or progressive rendering when appropriate.

## Reactivity Overhead

Vue deep reactivity is ergonomic for normal state. For very large immutable structures or external objects:

```ts
const rows = shallowRef<Row[]>([])
const graph = shallowRef<GraphModel | null>(null)
```

Rules:

- Treat nested data as immutable.
- Replace the root value to trigger updates.
- Use `markRaw()` for external class instances that should remain unmanaged by Vue.
- Document why shallow reactivity is used.

## Layout Thrash

Layout thrash pattern:

```txt
read layout -> write style -> read layout -> write style
```

Examples of layout reads:

```txt
getBoundingClientRect
offsetWidth / offsetHeight
clientWidth / clientHeight
scrollTop / scrollHeight
computed style reads
```

Rules:

- Batch reads before writes.
- Use `requestAnimationFrame` for visual measurement/update cycles.
- Use ResizeObserver/IntersectionObserver carefully and clean up.
- Prefer CSS layout over JS measurements when possible.
- Avoid measuring during every keypress, scroll, or mousemove without throttling.

## CSS Performance

Review changes that affect:

```txt
- layout: width, height, padding, margin, border, display, position, top/left
- paint: color, background, box-shadow, filter
- compositing: transform, opacity
```

Guidelines:

- Prefer transform/opacity for animations.
- Limit `will-change` to actual short-lived animations.
- Avoid expensive box shadows/filters in huge lists.
- Keep transitions scoped to intended properties.

## Load Performance

Check when adding heavy UI or dependencies:

```txt
- Does it load on app startup?
- Can it be lazy loaded by route or interaction?
- Does the dependency tree-shake?
- Is there an existing utility/component already loaded?
- Does Electron startup time matter for this path?
```

Use dynamic imports for heavy, rare surfaces.

```ts
const SettingsDialog = defineAsyncComponent(() => import('./SettingsDialog.vue'))
```

## Electron Renderer Performance

Renderer performance can degrade through IPC and main-process work.

Check:

```txt
- IPC call frequency per interaction:
- Payload size:
- Serialization cost:
- Main handler latency:
- Renderer blocked while waiting:
- Error/cancel path:
```

Prefer optimistic UI, cancellation, batching, and background work for slow operations.

## Performance Review Before Final

```txt
- DOM node count growth is intentional:
- Component instance count growth is intentional:
- Reactive source/watch count growth is intentional:
- Hot paths avoid broad store subscriptions:
- Large lists are virtualized or bounded:
- Layout read/write cycles are batched:
- IPC frequency and payload size are controlled:
- Profiling or reasoning supports the chosen approach:
```
