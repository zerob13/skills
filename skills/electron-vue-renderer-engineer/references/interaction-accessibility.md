# Interaction and Accessibility

A UI change is complete when interaction, keyboard, focus, semantics, and feedback remain correct.

## Interaction Map

For behavior changes, map:

```txt
Interaction Map
- Entry point:
- Primary action:
- Secondary/cancel action:
- Keyboard path:
- Focus before action:
- Focus after action:
- Loading state:
- Disabled state:
- Error state:
- Empty state:
- Screen reader semantics:
- Reduced motion/theme concerns:
```

## Focus Ownership

Focus behavior belongs with the interactive surface or its owner.

Check:

```txt
- What receives focus when opened?
- Where does focus return when closed?
- Can keyboard users reach all actions?
- Can focus escape a modal while it is open?
- Does route navigation restore focus to a meaningful place?
- Does async rendering require nextTick before focus?
```

Example:

```ts
const triggerRef = ref<HTMLElement | null>(null)
const closeButtonRef = ref<HTMLElement | null>(null)

async function openDialog() {
  isOpen.value = true
  await nextTick()
  closeButtonRef.value?.focus()
}

function closeDialog() {
  isOpen.value = false
  triggerRef.value?.focus()
}
```

## Dialog Interaction Checklist

```txt
- role="dialog" or native dialog semantics:
- aria-modal="true" for modal surfaces:
- accessible label via aria-labelledby or aria-label:
- initial focus:
- focus trap:
- escape key:
- backdrop/outside click policy:
- close button:
- focus restore:
- scroll lock:
- inert/background interaction policy:
- reduced motion respected:
```

Prefer existing modal/dialog primitives because they usually bundle these responsibilities.

## Button and Form Rules

- Use semantic `button`, `input`, `label`, `select`, and `textarea` elements where possible.
- Specify `type="button"` for non-submit buttons inside forms.
- Connect labels to controls.
- Represent disabled and loading states clearly.
- Keep validation messages connected to fields.
- Preserve submit-on-enter and escape/cancel conventions when the app uses them.

## Keyboard Behavior

Check common keys:

```txt
Enter
- submit primary action
- open selected item
- activate focused control

Escape
- close transient surfaces
- cancel edit mode
- clear search only when convention exists

Arrow keys
- menus, listboxes, tabs, trees, comboboxes

Tab / Shift+Tab
- focus order
- focus trap for modal surfaces
```

## Click Outside

For click-outside behavior:

```txt
- Define which surface owns outside detection.
- Ignore clicks inside Teleported content.
- Clean up document listeners.
- Handle pointerdown vs click deliberately.
- Preserve nested overlay behavior.
```

## Loading, Empty, and Error States

Every async UI path needs a clear state model:

```txt
idle -> loading -> success
idle -> loading -> error
success -> refreshing -> success/error
```

Check:

```txt
- Is the action disabled while unsafe to repeat?
- Is optimistic UI safe?
- Is stale async result ignored?
- Is retry available where useful?
- Is error text visible and actionable?
```

## Reduced Motion and Theme

When changing animation or visual state:

```css
@media (prefers-reduced-motion: reduce) {
  .panel {
    transition: none;
  }
}
```

Check dark/light/high-contrast tokens if the project supports them.

## Accessibility Review Before Final

```txt
- Semantic elements remain correct:
- Keyboard path works:
- Focus behavior works:
- Screen reader labels are present for controls/surfaces:
- Loading/disabled/error states are clear:
- Reduced motion/theme behavior is preserved:
```
