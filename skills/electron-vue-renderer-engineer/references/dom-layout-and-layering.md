# DOM, Layout, and Layering

Vue component ownership and actual DOM placement can differ. Layout bugs usually require both trees.

## Two Trees to Map

```txt
Vue logical tree
App
└─ AppShell
   └─ RouteView
      └─ FeaturePanel
         └─ ConfirmDialog  <- state owner / event owner

Actual DOM tree
body
├─ #app
│  └─ .app-shell
│     └─ .route-view
│        └─ .feature-panel
└─ #overlay-root
   └─ .dialog-layer
      └─ .confirm-dialog  <- visual layer owner
```

A Teleport keeps logical ownership while moving rendered DOM.

## Required Layout Map

For visual, position, overflow, stacking, scroll, or resize work:

```txt
Layout Map
- Rendered element:
- Logical owner:
- Actual DOM parent:
- Scroll container:
- Overflow ancestors:
- Stacking context ancestors:
- Positioning context:
- Transform/filter/perspective ancestors:
- Theme/token source:
- Existing utility/component:
```

## Stacking Context Checklist

Inspect ancestors for:

```txt
- position + z-index
- transform
- filter
- perspective
- opacity < 1
- contain
- isolation
- will-change
- mix-blend-mode
- overflow clipping
- fixed/sticky container behavior
```

A high `z-index` inside a constrained stacking context may still render below another layer.

## Overlay Policy

```txt
Viewport-level overlay
- modal
- full-screen dialog
- drawer
- global toast
- app-level command palette
- blocking loading mask

Use:
- Teleport to body or shared overlay root
- fixed positioning
- central z-index scale
- focus management
- escape/outside-click behavior
- scroll lock when modal

Anchor-level overlay
- dropdown menu
- popover
- tooltip
- combobox listbox
- context menu near pointer

Use:
- anchor-aware positioning
- viewport collision handling
- resize/scroll updates
- local or shared floating utility
```

## Positioning Policy

```txt
fixed
- viewport-level surfaces
- global overlays
- detached panels tied to window viewport

absolute
- local positioned container with deliberate ownership
- anchor content inside a known positioning context

sticky
- scroll-region headers or sidebars
- confirm scroll container ownership first

static/flex/grid
- normal layout flow
- preferred for structural page layout
```

Use structural layout first. Use offsets only when the chosen positioning model requires them.

## Teleport Policy

Use Teleport when visual DOM placement must escape a parent.

Check:

```txt
- Does the target element exist before Teleport mounts?
- Is there an existing #overlay-root, #modal-root, or body target?
- Does the component remain a logical child of its owner?
- Do props/emits/inject continue to represent ownership correctly?
- How are multiple Teleports stacked or ordered?
```

Common root markup:

```html
<body>
  <div id="app"></div>
  <div id="overlay-root"></div>
</body>
```

Component pattern:

```vue
<Teleport to="#overlay-root">
  <AppDialog v-if="open" @close="open = false" />
</Teleport>
```

## Dialog / Modal Checklist

```txt
- Open state owner:
- Close event owner:
- DOM mount target:
- Stacking order:
- Backdrop behavior:
- Focus initial target:
- Focus restore target:
- Escape key behavior:
- Outside click behavior:
- Scroll lock:
- Route change cleanup:
- Transition cleanup:
```

## Scroll and Overflow

Before changing overflow:

```txt
- Identify the intended scroll container.
- Identify sticky/fixed children.
- Check body scroll lock interactions.
- Check nested scroll regions.
- Check keyboard and wheel behavior.
- Check resize and small viewport behavior.
```

## CSS Token and Theme Rules

- Use existing tokens for z-index, spacing, radius, color, shadow, typography, and duration.
- Use semantic tokens for UI meaning.
- Keep raw values local only when they are intrinsic to a component primitive.
- Preserve dark/light/high-contrast variants when the project supports them.

## Layout Patch Review

Before finishing:

```txt
- Vue logical owner remains correct:
- DOM mount location matches visual requirement:
- Stacking context is intentional:
- Scroll container is intentional:
- Resize behavior is checked:
- Keyboard/focus behavior is checked for interactive overlays:
- Arbitrary z-index/offset growth is avoided:
```
