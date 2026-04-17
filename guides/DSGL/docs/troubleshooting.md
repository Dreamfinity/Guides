# Troubleshooting

This page focuses on failures and confusing behaviour that are common in the current DSGL runtime.

## Hooks: runtime errors and identity pitfalls

### Error: hook usage outside active render

Typical message:

- `Hook usage outside active component render is not allowed.`
- `Hook APIs require a UiScope owned by a DsglWindow render session.`

Why it happens:

- hook calls are not inside an active `DsglWindow.render()` -> `ui { ... }` session

What to do:

- call hooks only inside render composition scope
- avoid top-level `ui { ... }` usage for hook code

### Error: storage-backed hook used without `by`

Typical message:

- `Storage-backed hook 'useState' must be bound via delegated property syntax`

Why it happens:

- `useState`, `useMemo`, `useCallback`, and `useRef` are storage-backed delegate hooks

What to do:

```kotlin { .kotlin .copy .select }
// good
var count by useState(0)

// invalid in current runtime
val stateCell = useState(0)
```

### Error: hook signature mismatch / duplicate identity

Typical messages:

- `Hook signature mismatch at path ...`
- `Duplicate hook path '...' in component '...'`
- `Duplicate component identity ... in a single render pass.`

Why it happens:

- same hook path is reused with incompatible type/signature across renders
- the same hook path is resolved more than once inside one real component instance body
- duplicate `nodeKey`/identity is reused in one render (common in DnD hook setups)

What to do:

- keep hook call structure deterministic per component
- for repeated sibling custom components, keep item/component identity stable (keys where available)
- for unkeyed repeated siblings, expect ordinal fallback and position-based state rebinding on reorder
- keep keys stable and unique per rendered instance
- avoid conditional branches that reuse one delegated hook name for incompatible types

For full runtime rules, see [Hooks](hooks.md).

## State changes but UI does not rebuild

### Case 1: assigning the same value

`state(...)` and `useState(...)` invalidate only when new value differs from current value.
Assigning equal value is a no-op by design.

### Case 2: mutating object/ref in place

Common issue:

- mutate a mutable object/list in place, but do not publish a new state value

What to do:

- prefer immutable-style updates (`old + newItem`, `old.filterNot { ... }`, etc.)
- publish a new value through state setter/delegate assignment

### Case 3: `useRef` changes expected to rerender

`ref.current` updates do not trigger invalidation.
Use state for render-driving values; use refs for imperative handles and transient mutable data.

See [State and reactivity](state-reactivity.md) for exact rebuild semantics.

## Stylesheet changes do not apply

Checklist for MC 1.7.10 host:

1. DSS files are under `<mcDataDir>/dsgl/styles`.
2. File extension is `.dss`.
3. Parse logs do not show `[DSGL-Style] Parse error ...`.
4. Manual reload path was triggered (`F6` in `DsglScreenHost`, or `StyleEngine.forceReloadStylesheets()`).

If rules still do not affect node:

- confirm selector targets DSGL ids/classes/types that actually exist
- remember inline style has higher precedence than stylesheet rules
- confirm pseudo-state (`:hover`, `:active`, etc.) is actually active at runtime

## Overlay / popup input surprises

Input routing is layered, not flat:

1. system overlay
2. application overlay
3. application DOM tree

Practical effect:

- when a select/context menu/modal consumes pointer input, underlying app element will not receive that event
- this is expected behaviour, not click-through failure

Context menu caveat in current API:

- `onContextMenu` opens on right mouse down path and consumes that event path

## Clipping / scroll / positioning confusion

If pointer events seem to disappear:

- DSGL clips both paint and pointer input for clipped regions
- nested clips intersect

If `position: fixed` behaves unexpectedly:

- fixed nodes are viewport-anchored (root space), not ancestor-anchored
- they can escape ancestor overflow clips

If sticky does not move:

- sticky needs an inset (`top`, `bottom`, `left`, `right`) on the relevant axis
- behaviour is clamped by parent/scroll-container constraints

For current semantics (not browser assumptions), see [Layout model](layout-model.md).

## Hot-reload caveats

Hot reload depends on external signaling (`HotReloadBridge.markHotSwap()`), usually from the agent integration path.
No signal means no hot-swap rebuild.

During hot reload, incompatible hook signatures trigger remount/reset for affected component subtree instead of
normal-runtime crash.
This can reset hook-local state for that subtree.

Host behaviour in `DsglScreenHost`:

- hot-reload render attempts are retried (up to 8 attempts) when remount is requested
- if remount recovery never stabilizes, render attempt fails and previous committed frame is kept

For setup details, see [Hot-reload agent setup](hot-reload-agent.md).

