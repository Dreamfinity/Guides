# Hooks

## Before the API list

Hook behaviour depends on the window render lifecycle and the per-window hook runtime

Practical rules:

- Hook APIs are valid only inside a `UiScope` that belongs to a `DsglWindow` render session
- In practice, call hooks from `ui { ... }` inside `DsglWindow.render()`
- Calling hook APIs outside an active render session fails with a runtime error
- Calling hook APIs from a `UiScope` not owned by a window (for example top-level `ui { ... }`) fails with:
  `Hook APIs require a UiScope owned by a DsglWindow render session

## Runtime ordering and identity rules

These rules are enforced by `ComponentHookRuntime` and tests:

- Hook state is keyed by component identity + hook path inside that component
- Hook slots themselves stay path-based (delegated name/custom-scope path/synthetic unnamed path)
- Sibling custom `UiScope` components are auto-scoped into distinct inferred component instances when no explicit
  runtime scope is present
- Explicit runtime component scopes (`withComponentInstance(...)`) take precedence over inferred fallback
- For unkeyed inferred component siblings from the same invocation site, instance identity falls back to ordinal
  position in that parent scope
- Reordering unkeyed siblings is position-sensitive: state follows position, not semantic item identity
- For storage-backed delegated hooks, hook path comes from delegated property name
- For unnamed hooks (`useReducer`, `useEffect`, `useEffectEveryCommit`), hook path is synthetic (`useReducer#0`,
  `useEffect#0`, ...), so call order inside a scope matters
- Duplicate hook paths inside one real component instance still fail loudly
- Reusing the same hook path with a different kind/signature throws in normal runtime
- In hot-reload mode, incompatible paths trigger subtree remount/reset instead of keeping stale hook state
- Hook state/effects are dropped when that component subtree disappears from the DOM tree and reinitialized on
  reappearance

### Hooks that require delegated `by` usage

These are storage-backed and must be bound with delegated syntax:

- `useState`
- `useMemo`
- `useCallback`
- `useRef`

Direct assignment (without `by`) fails at render end with `HookUsageException`.

## State and computation

### `useState`

What it is for:

- Local mutable state for a component

Usage shape:

```kotlin
fun UiScope.counterBlock() {
    var count by useState(0)
    button("Increment", { onMouseClick = { count += 1 } })
    text("count=$count")
}
```

Delegate-backed:

- Yes, requires `by`

Persistence across rebuilds:

- Persists for the same component identity and hook path.
- Resets when the component subtree disappears and later reappears.

Caveats:

- Writing the same value does not trigger invalidation.
- Using incompatible `useState` types on the same path throws in normal runtime.

### `useReducer`

What it is for:

- Local state transitions via reducer + dispatch.

Usage shape:

```kotlin
fun UiScope.counterReducer() {
    val (count, dispatch) = useReducer(0) { old: Int, action: Int -> old + action }
    button("+5", { onMouseClick = { dispatch(5) } })
    text("count=$count")
}
```

Delegate-backed:

- No

Persistence across rebuilds:

- Reducer state persists for the same component identity and hook path.
- Resets after subtree disappearance/reappearance.

Caveats:

- Dispatch triggers rebuild only when reducer returns a different value.
- Incompatible reducer signature on the same path throws in normal runtime.

### `useMemo`

What it is for:

- Cache derived values between rebuilds.

Usage shape:

```kotlin
fun UiScope.derivedText(input: String) {
    val label by useMemo(input) { "Derived: ${input.uppercase()}" }
    text(label)
}
```

Delegate-backed:

- Yes, requires `by`

Persistence across rebuilds:

- Cached value persists for the same component identity/path.
- Recomputes when ordered dependency list changes.
- `useMemo { ... }` without deps computes once per mounted hook instance.

Caveats:

- Resets after subtree disappearance/reappearance.
- Incompatible type signature on same path throws in normal runtime.

### `useCallback`

What it is for:

- Cache callable objects/functions with dependency-based identity changes.

Usage shape:

```kotlin
fun UiScope.callbackSample(dep: Int) {
    val callback by useCallback(dep) {
        val captured = dep
        { captured }
    }
    text("callback()=${callback()}")
}
```

Delegate-backed:

- Yes, requires `by`

Persistence across rebuilds:

- Function identity stays stable while deps stay equal.
- New function object is produced when deps change.

Caveats:

- Same storage/delegate rules as `useMemo`.

## Effects

### `useEffect`

What it is for:

- Post-commit side effects with dependency-based rerun and cleanup.

Usage shape:

```kotlin
fun UiScope.syncSomething(dep: String, log: MutableList<String>) {
    useEffect(dep) {
        log += "run:$dep"
        onDispose { log += "cleanup:$dep" }
    }
}
```

Delegate-backed:

- No

Persistence across rebuilds:

- Last committed effect state/cleanup is tracked by hook runtime per component/path.

Caveats:

- Effect body runs only after successful `commitRenderBuild()`.
- Dependency-change cleanup runs before rerun.
- If render is discarded/aborted, effect body and rerun cleanup do not run for that attempt.
- Cleanup runs on subtree disappearance and on `disposeHookRuntime()`.

### `useEffectEveryCommit`

What it is for:

- Post-commit side effect that reruns on every successful commit.

Usage shape:

```kotlin
fun UiScope.trackCommits(log: MutableList<String>) {
    useEffectEveryCommit {
        log += "run"
        onDispose { log += "cleanup" }
    }
}
```

Delegate-backed:

- No.

Persistence across rebuilds:

- The effect slot persists by path; cleanup + rerun happens every successful commit.

Caveats:

- Same commit/discard rules as `useEffect`.

## Refs and handles

### `useRef`

What it is for:

- Mutable cell that survives rebuilds without being itself a rebuild trigger.

Usage shape:

```kotlin
fun UiScope.refSample() {
    val countRef by useRef<Int>()
    countRef.current = (countRef.current ?: 0) + 1
    text("seen=${countRef.current}")
}
```

Delegate-backed:

- Yes, requires `by`.

Persistence across rebuilds:

- `Ref` object persists for the same component identity/path.
- Resets when subtree disappears and later reappears.

Caveats:

- Updating `ref.current` does not invalidate the window by itself.

### `createRef`

What it is for:

- Non-hook ref object creation when you need a `Ref<T>` without hook runtime identity.

Usage shape:

```kotlin
val externalRef = createRef<String>()
externalRef.current = "value"
```

Delegate-backed:

- No.

Persistence across rebuilds:

- Lifetime is normal object lifetime, not hook-managed.

### `ElementHandle`

`ElementHandle` is the runtime handle type usually stored in refs attached to elements (`ComponentProps.ref`).

Available operations:

- `key`
- `bounds`
- `requestFocus()`
- `scrollIntoView()`

Current caveat:

- `scrollIntoView()` is currently a TODO/no-op in implementation.

## Context

### `createContext`

What it is for:

- Define a typed context key with default value.

Usage shape:

```kotlin
private val ThemeContext = createContext(defaultValue = "System", name = "Theme")
```

Delegate-backed:

- No.

Persistence across rebuilds:

- Context object is a regular value you keep in your own scope.

### `provideContext`

What it is for:

- Provide a context value to a nested composition subtree.

Usage shape:

```kotlin
fun UiScope.themeArea(theme: String) {
    provideContext(ThemeContext, theme) {
        text("theme=${useContext(ThemeContext)}")
    }
}
```

Delegate-backed:

- No.

Persistence across rebuilds:

- Provider value is reapplied each rebuild from current composition logic.

Caveats:

- Provider scope is lexical in current composition, not global.

### `useContext`

What it is for:

- Read nearest provided value for a context key.

Usage shape:

```kotlin
val currentTheme = useContext(ThemeContext)
```

Delegate-backed:

- No.

Persistence across rebuilds:

- Reads nearest provider each rebuild.
- Falls back to `defaultValue` if no provider exists.
- If nested providers exist, nearest provider wins.

## DnD-related hooks and utilities

These are public and usable, but they are more specialized than core state/effect hooks.

### `useDraggable`

What it is for:

- Create draggable descriptor/state for one element.

Usage shape:

```kotlin
val draggable = useDraggable(id = "card-a", nodeKey = "card-a")
div({
    applyDraggable(draggable)
}) {
    text("Card A")
}
```

Delegate-backed:

- No.

Persistence across rebuilds:

- Descriptor is rebuilt each render.
- Drag state is managed by DnD runtime + hook runtime identity (`nodeKey`).

Caveats:

- Duplicate component identity (same hook/component identity in one render) throws.

### `useDroppable`

What it is for:

- Create drop target descriptor/state for one element.

Usage shape:

```kotlin
val droppable = useDroppable(id = "lane-a", nodeKey = "lane-a")
div({
    applyDroppable(droppable)
}) {
    text("Lane A")
}
```

Delegate-backed:

- No.

Persistence across rebuilds:

- Descriptor is rebuilt each render; over/active status comes from DnD runtime state.

### `useSortable`

What it is for:

- Compose draggable + droppable behaviour for sortable lists/containers.

Usage shape:

```kotlin
val sortable = useSortable(
    id = itemId,
    nodeKey = itemId,
    containerId = "lane-a",
    items = items
)
div({
    applySortable(sortable)
}) {
    text(itemId)
}
```

Delegate-backed:

- No.

Persistence across rebuilds:

- Sortable container state persists while mounted and resets after disappearance.

Caveats:

- Sortable internals are intentionally DSGL-specific; keep usage at the descriptor level (`useSortable` +
  `applySortable`).

### `useDragDropMonitor`

What it is for:

- Subscribe to drag-drop monitor callbacks in composition scope.

Usage shape:

```kotlin
useDragDropMonitor(
    DragDropMonitorCallbacks(
        onDragEnd = { active, over, effect ->
            // react to drag end
        }
    )
)
```

Delegate-backed:

- No.

Persistence across rebuilds:

- One subscription is retained while mounted.
- Callback closures are refreshed on rebuild.

Caveats:

- Cleanup/unsubscribe runs on disappearance and hook runtime disposal.

### `reorderByDnD` utility

`reorderByDnD(...)` is a pure helper for list reordering. It is not a hook and does not require render-session usage.

### `applyDraggable`, `applyDroppable`, `applySortable` utilities

What they are for:

- Attach DnD descriptors to element/component props.

Usage shape:

```kotlin
val draggable = useDraggable(id = "card-a")
val droppable = useDroppable(id = "lane-a")

div({
    applyDraggable(draggable)
    applyDroppable(droppable)
}) {
    text("Card A")
}
```

Delegate-backed:

- No (plain `ComponentProps` helpers).

Persistence across rebuilds:

- They are stateless prop-wiring helpers; persistence comes from the underlying hook/runtime descriptors.

## Practical checklist

- Keep hook calls deterministic for each component instance.
- Use stable keys/component identity where you expect state persistence.
- For repeated unkeyed sibling component calls, expect ordinal fallback semantics and position-based rebinding on
  reorder.
- Use `by` for storage-backed hooks.
- Treat DnD hooks as advanced descriptor APIs, not as a generic public overlay framework.
- For runtime behaviour details around rebuild triggers, see [State and reactivity](state-reactivity.md).

