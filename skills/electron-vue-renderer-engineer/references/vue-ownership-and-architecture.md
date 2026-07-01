# Vue Ownership and Architecture

Vue renderer work should preserve ownership clarity.

## Ownership Layers

```txt
App shell
└─ Layout
   └─ Route view
      └─ Feature container
         └─ UI component
            └─ Primitive component
```

Typical responsibilities:

```txt
App shell
- global providers
- app-level layout
- global overlay root
- global keyboard/window integrations
- app readiness and bootstrap concerns

Layout
- persistent navigation/sidebar/header/footer
- responsive regions
- route outlet placement
- layout-level scrolling and overflow

Route view
- route params/query integration
- route-level data loading
- feature composition
- route guard/result handling

Feature container
- feature state coordination
- async feature operations
- orchestration between child components

UI component
- focused UI responsibility
- typed props/emits
- local ephemeral state
- visual rendering

Primitive component
- reusable visual/interaction primitive
- stable API
- theme/token compatibility
```

## Locate the Logical Owner

For any change, identify:

```txt
- Who decides whether it exists?
- Who decides what value it shows?
- Who handles user action?
- Who owns loading/error/disabled state?
- Who owns navigation or IPC side effects?
- Who already owns similar behavior nearby?
```

Patch the logical owner, then let rendering components remain simple.

## Component Boundary Rules

- Keep root, app shell, layout, and route components as composition surfaces.
- Keep feature coordination in feature containers or composables.
- Keep presentational components focused on rendering typed inputs.
- Keep primitives generic and reusable.
- Use props down and events up for normal parent-child contracts.
- Use `provide`/`inject` for deep local context where prop drilling creates noise and the scope remains bounded.
- Use stores for cross-route, cross-window, persistent, or domain state.
- Use slots for layout/composition flexibility while preserving ownership at the caller.

## Component Split Signals

Split or extract when:

- One SFC mixes route loading, store mutation, layout, form control, and primitive rendering.
- A block has a nameable responsibility and repeated dependency set.
- A side effect can be reused or tested separately.
- The template hides important state transitions.
- Multiple unrelated concerns are forced to re-render together.

Keep inline when:

- The logic is short, local, and has no reuse path.
- Extraction would create prop/event noise larger than the removed code.
- The component is in a hot list and abstraction cost matters.

## Props and Emits

Use typed props and emits.

```ts
const props = defineProps<{
  modelValue: string
  disabled?: boolean
  loading?: boolean
}>()

const emit = defineEmits<{
  'update:modelValue': [value: string]
  submit: []
  cancel: []
}>()
```

Guidelines:

- Keep props stable in frequently updated parents.
- Pass derived booleans instead of full objects when the child only needs the boolean.
- Emit intent events from children; perform business effects in owners.
- Keep component APIs narrow.

## Slots

Slots move rendering responsibility to the caller.

Before editing slotted content:

```txt
- Locate the slot provider.
- Locate the slot consumer.
- Determine which side owns layout.
- Determine which side owns behavior.
- Preserve slot prop contracts.
```

## Dynamic Components / KeepAlive / Suspense / Transition

These built-ins can hide lifecycle or DOM behavior.

Check:

```txt
- Is the target component mounted, cached, suspended, or transitioning?
- Does state survive route/component switches?
- Does cleanup happen on unmount, deactivation, or transition leave?
- Does layout measurement happen before DOM is stable?
```

## File Placement

Follow existing project layout. Common patterns:

```txt
src/
  app/ or layouts/
  pages/ or views/
  features/<feature>/
    components/
    composables/
    stores/
    types.ts
  shared/
    components/
    composables/
    utils/
```

Use existing folders and naming over introducing a new architecture.
