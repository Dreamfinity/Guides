# Custom Components

## What a custom component is in DSGL?

- a reusable `UiScope` extension function
- with parameters for data/configuration (e.g. props)
- optionally with callbacks and content slots
- built from public DSL elements (`div`, `text`, `button`, `input`, `select`, ...)

Example:

```kotlin { .kotlin .copy .select }
fun UiScope.statusBadge(
    label: String,
    color: Int
) {
    div({
        key = "status.badge.$label"
        style = {
            display = Display.Block
            padding = 2.px
            backgroundColor = color
        }
    }) {
        text(label)
    }
}
```

This is the same composition style used by DSGL helper DSLs in core code (`modal*` helpers, `selectModel`, `contextMenu`
builders).

## Data/configuration patterns that work well

### 1) Stateless + callback

Use this as the default for reusable UI:

```kotlin { .kotlin .copy .select }
fun UiScope.counterRow(
    title: String,
    value: Int,
    onIncrement: () -> Unit
) {
    div({
        key = "counter.row.$title"
        style = {
            display = Display.Flex
            flexDirection = FlexDirection.Row
            gap = 4.px
        }
    }) {
        text("$title: $value")
        button("+1", { onMouseClick = { onIncrement() } })
    }
}
```

- state ownership stays explicit in the caller
- easy to reuse in different windows
- easy to test and reason about

### 2) Slot-based composition

Use `UiScope.() -> Unit` slots when callers need custom content:

```kotlin { .kotlin .copy .select }
fun UiScope.panel(
    title: String,
    keySuffix: String,
    body: UiScope.() -> Unit
) {
    div({
        key = "panel.$keySuffix"
        style = {
            display = Display.Flex
            flexDirection = FlexDirection.Column
            gap = 3.px
            padding = 4.px
        }
    }) {
        text(title)
        body()
    }
}
```

## State placement: window vs component hooks

Use window-owned state (`DsglWindow.state(...)`) when state is shared across multiple components, survives broader
screen changes, or should remain centrally controlled

Use hook-local state inside a custom component (`useState` / `useReducer`) when state is truly local to that
reusable piece

```kotlin { .kotlin .copy .select }
fun UiScope.expandableNote(title: String, note: String) {
    var expanded by useState(false)

    div({
        key = "expandable.$title"
        style = {
            display = Display.Flex
            flexDirection = FlexDirection.Column
            gap = 2.px
        }
    }) {
        button(if (expanded) "Hide" else "Show", {
            onMouseClick = { expanded = !expanded }
        })
        if (expanded) {
            text("$title: $note")
        }
    }
}
```

Hook constraints still apply:

- hook calls must run in a `UiScope` owned by a `DsglWindow` render session
- storage-backed hooks require delegated `by` syntax

See [Hooks](hooks.md) and [State and reactivity](state-reactivity.md) for runtime details.

## Key and identity guidance for reusable interactive components

In practice:

- put a stable `key` on the component root element when instances can move or reorder
- when rendering lists, derive the key from stable domain id, not index
- keep key format predictable (`componentName.itemId`)

Why this matters:

- DOM reconciliation uses keys for element identity
- focus retention also relies on key/path fallback

Important hook-runtime caveat:

- Hook slots are path-based and duplicate hook paths in the same real component instance fail at runtime.
- Sibling hook-using custom `UiScope` components are automatically separated into distinct component-instance scopes.
- If no explicit instance scope/key exists, repeated siblings from the same invocation site use ordinal fallback
  identity.
- Reordering unkeyed siblings is intentionally position-sensitive (state follows position).
- Some advanced DSGL utilities (for example, DnD hooks) create internal component scopes themselves.
- For your own reusable components, avoid relying on internal hook-runtime scoping APIs.

## What not to use as a public component authoring API

Avoid building app components around internal plumbing:

- `UiScope.mount(...)` (internal)
- `DsglWindow.hookRuntime()` and `ComponentHookRuntime.withComponentInstance(...)` (internal runtime plumbing)
- overlay/system internals and devtool internals (`overlay.system.*`, inspector internals, `HotReloadBridge`)

These exist for framework/runtime internals, not as stable public extension contracts.

## Practical checklist

- Start with `UiScope` extension + parameters.
- Prefer explicit data in and callbacks out.
- Add slot lambdas for extensibility.
- Keep the hook state local only when it is truly local.
- Use stable keys for interactive or reorderable component instances.
- Treat internal runtime/overlay APIs as non-public unless docs explicitly promote them.

Next: [Hot-reload setup](hot-reload-agent.md).
