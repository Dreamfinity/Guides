# Layout Model

## Layout pipeline in practice

At runtime, the layout is resolved in this order:

1. resolve computed style
2. resolve style lengths into numeric node fields
3. measure nodes (`measure` / `measureForLayout`)
4. place nodes (`render`)
5. build paint commands (`appendRenderCommands`)
6. route input using the same used geometry model

That render/input symmetry is important for positioned and clipped nodes.

## Display Modes

### `display: block`

`ContainerNode` block flow supports mixed block and inline-formatting children:

- block children are stacked vertically
- inline/text children form line boxes and wrap when width is constrained
- `gap` is applied between rows/lines in DSGL terms

### `display: inline`

Inline container mode lays children left-to-right with wrapping.

- wrapping happens against the available content width
- line height uses measured child outer height plus resolved line-height constraints
- dedicated tests cover no-overlap behaviour for mixed nested children

### `display: flex`

Current flex behaviour is single-line row/column distribution:

- axis from `flex-direction`
- size distribution with `flex-grow`/`flex-shrink`/`flex-basis`
- main-axis alignment from `justify-content`
- cross-axis alignment from `align-items`
- no public `flex-wrap` style property in the current model

### `display: grid`

Current grid behaviour is numeric track-based:

- `grid-columns` sets column count
- `grid-rows` is optional; auto placement can expand rows
- `grid-auto-flow` controls row-first vs column-first placement
- item spans from `grid-column-span` / `grid-row-span`
- per-cell alignment uses `justify-items` + `align-items`

This is intentionally narrower than browser CSS grid.

### `display: none`

Node is excluded from layout/render traversal and state like hover/active/focus/open is reset for that node.

### Overlay stack container (DSL helper)

`overlay { ... }` uses a `ContainerNode` with `stackLayout=true`.

- children share the same content area
- child `align` controls placement inside that area
- this is a DSGL container mode, not a CSS `position` feature

Example:

```kotlin { .kotlin .copy .select }
overlay({
    key = "hud.layer"
    style = {
        width = 100.percent
        height = 100.percent
    }
}) {
    div({
        key = "badge"
        style = {
            align = StyleAlign.END
            margin(8.px)
            padding = 4.px
        }
    }) {
        text("Pinned")
    }
}
```

## Sizing and Units

DSGL resolves style lengths via `CssLength` with units:

- `px`, `rem`, `em`, `vw`, `vh`, `%`

Percent bases are explicit in runtime:

- width-like values resolve against containing block width
- height-like values resolve against containing block height
- `em` resolves against current element computed font size
- `rem` resolves against root computed font size

Constraints:

- `min-*`/`max-*` clamp measured content sizes
- width/height values are clamped to non-negative values
- negative lengths are rejected for width/height/border/gap/font-size-style paths

## Positioning Modes

### `static`

- default mode
- ignores offsets (`left/top/right/bottom`) for used geometry
- stays in normal flow
- `z-index` does not reorder static siblings

### `relative`

- stays in normal flow
- applies visual/input offset after normal slot placement
- precedence rules: `left` over `right`, `top` over `bottom`

### `absolute`

- removed from normal flow
- containing block: nearest positioned ancestor (`relative`/`absolute`/`fixed`) or root
- offset precedence is the same (`left` over `right`, `top` over `bottom`)

### `fixed`

- removed from normal flow
- anchored to root viewport, not nearest positioned ancestor
- participates in root ordering (promotion model)
- escapes ancestor overflow clipping, but still clipped to root viewport

### `sticky`

- remains in normal flow (slot preserved)
- visual offset activates per axis only when that axis has an inset (`top`/`bottom`, `left`/`right`)
- inset precedence: top over bottom, left over right
- reference scroll container is nearest scroll-container ancestor per axis (fallback to root)
- movement is clamped by direct parent containing block

## Overflow and Scroll behaviour

Overflow is axis-based (`overflow-x`, `overflow-y`).

- `visible`: no scroll container behaviour on that axis
- `hidden`: clipped axis, no scrollbar
- `scroll`: always scrollbar on that axis
- `auto`: scrollbar when content exceeds viewport

Runtime properties covered by tests:

- clipping applies to both paint and pointer input (no click-through outside visible viewport)
- nested clips intersect
- mixed-axis cases (`visible` + non-visible on the other axis) still clip using resolved viewport behaviour
- scrollbar gutter reduces effective viewport size when present

Example:

```dss { .css .copy .select }
.panel {
  width: 260px;
  height: 140px;
  overflow-x: visible;
  overflow-y: auto;
}
```

<!-- Screenshot: layout overflow clip and scrollbars (images/layout-overflow-scroll.png) -->

## Z-order and Stacking

DSGL ordering uses one model for paint and hit-testing (hit order is reverse paint order).

Important rules from the current model:

- positioned nodes participate in positioned ordering buckets
- static nodes keep DOM-order behaviour
- positioned child with non-default `z-index` can act as child-context participant
- fixed descendants can be promoted to root ordering while preserving logical parent ownership

This is DSGL stacking behaviour, not full browser stacking-context parity.

## Known Limits (Current M4 Scope)

- No documented browser-equivalent `flex-wrap`, `align-content`, or advanced grid template syntax.
- Grid tracks are numeric and equal-width style in current container algorithm.
- Some CSS-like names exist with DSGL semantics only; do not assume browser parity unless tests/code confirm it.
- `display`/position/overflow behaviour is implementation-grounded and may differ from web engines in edge cases.
