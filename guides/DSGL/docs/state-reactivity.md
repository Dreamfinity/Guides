# State and Reactivity

## What is the state made for?

State allows you to keep track of UI-related data that is not directly related to the UI tree.

## How it works - shortly

- A window (`DsglWindow`) owns state and builds the DOM tree in `render()`
- A screen host (for example `DsglScreenHost`) asks the window to rebuild when invalidated
- Hook state is tracked by the per-window hook runtime

## Two state models

### Window-owned state: `state(...)`

Use this for a state owned by the whole window:

```kotlin { .kotlin .copy .select }
import org.dreamfinity.dsgl.core.DsglWindow
import org.dreamfinity.dsgl.core.dsl.UiScope
import org.dreamfinity.dsgl.core.dsl.button
import org.dreamfinity.dsgl.core.dsl.text
import org.dreamfinity.dsgl.core.event.Event
import org.dreamfinity.dsgl.mcForge1710.demo.examples.containers.centeredFlexWrapper

class GlobalStateWindow : DsglWindow() {
    private var counter by state(0)

    override fun render() = ui {
        centeredFlexWrapper {
            globalStateCounter(counter, { counter += 1 })
        }
    }
}

private fun UiScope.globalStateCounter(counter: Int, setCounter: (_: Event) -> Unit) {
    button("Increment", { onMouseClick = setCounter })
    text("Counter: $counter")
}
```

![img](images/global-state-counter.png)

Behaviour in the current implementation:

- `state(...)` returns a delegate-backed mutable state holder
- Setting a different value triggers `invalidate()`
- With an attached host, `invalidate()` requests rebuild via `requestRebuild("state")`
- Setting the same value does not request rebuild

This might be useful for window-wide state not tied to a specific component until a
global state manager is implemented

### Hook state: `useState(...)` and `useReducer(...)`

Use this for local state inside `UiScope` composition:

```kotlin { .kotlin .copy .select }
fun UiScope.counterCard() {
    var count by useState(0)
    val (sum, dispatch) = useReducer(0) { old: Int, delta: Int -> old + delta }

    button("count +1", { onMouseClick = { count += 1 } })
    button("sum +5", { onMouseClick = { dispatch(5) } })
    text("count=$count sum=$sum")
}
```

Behaviour in the current implementation:

- `useState` uses the same invalidation path as `state(...)` (via `window.onHookStateChanged()`)
- `useReducer` dispatch also uses that same path
- Hook local state persists across rebuilds for the same component identity
- If a hook subtree disappears and later reappears, the hook state is reinitialized

Important usage rule:

- Storage-backed hooks must be used with delegated syntax (`by`): `useState`, `useMemo`, `useCallback`, `useRef`
- Direct assignment (without `by`) fails at runtime with a hook usage error

## What triggers rebuilds

- `var x by state(...)` set to a different value
- `var x by useState(...)` set to a different value
- `useReducer` dispatch that changes reducer state
- Explicit `invalidate()` call from window code
- Host-driven rebuild requests (for example, resize / hot-reload handling in `DsglScreenHost`)

What does not trigger rebuild by itself:

- Reassigning the same value
- Mutating a mutable object in place without changing the delegated state value (because mutation is a bad UI patter,
  use reducers with clean functions instead)
- `useMemo`, `useCallback`, `useRef`, `useContext` reads alone
- `useEffect` registration alone (it runs after commit; rebuild happens only if effect code mutates state or calls
  invalidation through your code)

## Commit semantics for effects

`useEffect`/`useEffectEveryCommit` are render commit-bound:

- Effects run after a successful render commit
- Dependency change clean-up runs before rerun
- Unmount/disappearance triggers clean-up
- Aborted/failed render attempts to do not run effect bodies and do not run dependency-switch clean-up for that aborted
  attempt

This is why effect behaviour can differ from immediate render-time logic

## Best practices

- Keep window-wide state in `state(...)` (until global state manager is implemented)
- Keep component-local interactive state in `useState` / `useReducer`
- Use stable keys for interactive elements to preserve focus/state across rebuilds
- If you need a forced rebuild, expose a window method that calls `invalidate()` instead of wiring host APIs directly
  into components

Next: [Custom components](custom-components.md).