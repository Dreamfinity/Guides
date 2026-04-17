# Elements Overview

## Common element shape

Most `UiScope` element builders accept `ComponentProps` (or a derived props type), so these fields are broadly
available:

- identity and styling: `key`, `id`, `className`, `classes`, `style`
- state flags: `disabled`
- event handlers: `onMouse*`, `onKey*`, `onFocus*`, `onInput`, `onValueChange`

## Text and content

### `text(...)`

Use for static text or rebuild-time dynamic text.

Builder shape:

```kotlin { .kotlin .copy .select }
fun UiScope.textContentSample() {
    var clicks by useState(0)

    text("Literal")
    text("Literal", minecraftFormatting = true)
    dynamicText { "Clicks: $clicks" }
    button("Increment", { onMouseClick = { clicks += 1 } })
}
```

### `dynamicText(...)`

Convenience for dynamic text providers.

Builder shape:

```kotlin { .kotlin .copy .select }
dynamicText(
    value = { "runtime=${System.currentTimeMillis()}" },
    props = { style = { color = 0xFFEAF2FD.toInt() } }
)
```

### `button(...)`

Clickable text button.

Builder shape:

```kotlin { .kotlin .copy .select }
button("Apply", {
    onMouseClick = { /* your action */ }
})
```

## Containers and layout primitives

### `div(...)`

General-purpose container. Layout behaviour comes from style (`display`, `flexDirection`, etc).

Builder shape:

```kotlin { .kotlin .copy .select }
div({
    key = "panel"
    style = {
        display = Display.Flex
        flexDirection = FlexDirection.Column
        gap = 4.px
        padding = 4.px
    }
}) {
    text("Panel title")
}
```

### `overlay(...)`

Container for overlapping children (stack-like behaviour).

Builder shape:

```kotlin { .kotlin .copy .select }
overlay({
    key = "overlay.layer"
    style = { width = 100.percent }
}) {
    div({ style = { margin(8.px, 0.px, 0.px, 8.px) } }) { text("Floating badge") }
}
```

## Media and MC-specific content

### `img(...)`

Image node. Generic supported source paths:

- `file://...` (resolved under `<mcDataDir>/dsgl/...`)
- `http://` / `https://` (downloaded/cached under `<mcDataDir>/dsgl/cache/downloads/...`)

In MC 1.7.10 host, source also supports:

- resource id (`"minecraft:textures/..."`)

!!! info

    Custom domains support will be added in the future

Builder shape:

```kotlin { .kotlin .copy .select }
img("minecraft:textures/gui/options_background.png", {
    style = {
        width = 36.px
        height = 36.px
    }
})
```

### `itemStack(...)`

MC-facing element for item stack rendering through `ItemStackRef` adapters.

Builder shape (MC 1.7.10):

```kotlin { .kotlin .copy .select }
itemStack(McItemStackRef(itemStack), {
    size = 20
    rotYDeg = 160.0
    rotXDeg = -11.0
    style = { width = 70.px }
})
```

Caveat:

- for MC 1.7.10, use `McItemStackRef`; this is backend-specific integration, not a platform-agnostic game-item API.

## Inputs

### `input(InputType...)`

Single entry point for typed input elements.

Supported input variants right now:

- `InputType.Text`
- `InputType.Password`
- `InputType.Number`
- `InputType.Range`
- `InputType.Checkbox`
- `InputType.Radio`
- `InputType.Date`

!!! info

    Better DSL builder support is planned for future releases

Small example:

```kotlin { .kotlin .copy .select }
fun UiScope.inputSample() {
    var numberValue by useState(15L)
    var radioValue by useState<String?>("center")

    input(InputType.Number(value = numberValue, min = 0, max = 100), {
        key = "example.number"
        onValueChange = { event -> numberValue = event.value.toLongOrNull() ?: numberValue }
    })

    input(
        InputType.Radio(
            variants = listOf(InputOption("north", "North"), InputOption("center", "Center")),
            selected = radioValue
        ), {
            key = "example.radio"
            onValueChange = { event -> radioValue = event.parsedValue as? String }
        })
}
```

### `textarea(...)`

Dedicated multiline text input.

Builder shape:

```kotlin { .kotlin .copy .select }
textarea({
    key = "example.notes"
    placeholder = "Type multiple lines"
    style = { height = 40.px }
})
```

### `toggle(...)`

Switch-like boolean input with controlled/uncontrolled modes.

Builder shape:

```kotlin { .kotlin .copy .select }
var enabledFlag by useState(false)

toggle({
    key = "example.toggle"
    checked = enabledFlag
    onValueChange = { event -> enabledFlag = event.parsedValue as? Boolean ?: false }
})
```

Caveat:

- if `checked` is set, toggle acts as controlled; if omitted, `defaultChecked` is used.

### `select(...)`

Select field with popup list model built inline via `SelectModelBuilder`.

Builder shape:

```kotlin { .kotlin .copy .select }
var currentValue by useState<String?>(null)

select({
    key = "example.select"
    value = currentValue
    onValueChange = { event -> currentValue = event.value }
}) {
    placeholder("Choose one")
    option("apple", "Apple")
    separator()
    group("Citrus") {
        option("orange", "Orange")
        option("lemon", "Lemon") { enabled(false) }
    }
}
```

Caveats:

- popup behaviour is provided by select runtime/overlay internals; this API is the supported convenience entrypoint.
- keyboard and wheel behaviour are implemented and covered by `SelectEngineTests`.

### `colorPicker(...)` and `colorPickerPopup(...)`

Color picker primitives:

- `colorPicker(...)`: inline picker element
- `colorPickerPopup(...)`: field that opens picker popup

Builder shape:

```kotlin { .kotlin .copy .select }
var colorValue by useState(RgbaColor.WHITE)

colorPicker({
    key = "example.color.inline"
    value = colorValue
    onPreviewColor = { colorValue = it }
    onChangeColor = { colorValue = it }
    onCommitColor = { colorValue = it }
})

colorPickerPopup({
    key = "example.color.popup"
    value = colorValue
    popupCloseOnOutsideClick = false
    onCommitColor = { colorValue = it }
})
```

## Higher-level helper DSLs

These are exposed conveniences for common UI workflows. They are not documented as a generalized public overlay
framework contract.

### Modal helpers

Public helper set:

- `modalHost(modals, modalKey) { ... }`
- `ModalSpec(...)`
- `modalFrame`, `modalDialog`, `modalHeader`, `modalTitle`, `modalBody`, `modalFooter`
- `alertModal`, `confirmModal`, `promptModal`

Small example:

```kotlin { .kotlin .copy .select }
fun UiScope.modalSample() {
    var modals by useState(emptyList<ModalSpec>())

    modalHost(modals = modals, modalKey = "example.modal.host") {
        button("Open modal", {
            onMouseClick = {
                modals = listOf(
                    ModalSpec(key = "example.modal") { scope ->
                        modalHeader(closeButton = true, onHide = scope.dismiss) { modalTitle("Example") }
                        modalBody { text("Modal body") }
                        modalFooter {
                            button("Close", { onMouseClick = { scope.dismiss?.invoke() } })
                        }
                    }
                )
            }
        })
    }
}
```

Caveat:

- modal focus restore/trap/topmost handling is runtime-managed and tested (`ModalRuntimeTests`).

### Context menu helpers

Public helper set:

- model builder: `contextMenu { ... }`
- bind trigger: `DOMNode.onContextMenu { ... }` / `DOMNode.onContextMenuModel(...)`

Small example:

```kotlin { .kotlin .copy .select }
fun UiScope.contextMenuSample() {
    var lastAction by useState("none")

    val tileNode = div({ key = "file.tile" }) {
        text("Right-click me")
    }

    tileNode.onContextMenu {
        openMenu(
            contextMenu(id = "file.menu") {
                item("Open") { onClick { lastAction = "open" } }
                item("Rename") { onClick { lastAction = "rename" } }
                separator()
                item("Delete") { onClick { lastAction = "delete" } }
            }
        )
    }
}
```

Caveat:

- trigger currently opens on right mouse down and consumes the event in that path.
