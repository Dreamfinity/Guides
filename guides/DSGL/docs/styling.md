# Styling

## Where styles come from

### Inline DSL on elements

```kotlin { .kotlin .copy .select }
ui {
    div({
        key = "card"
        className = "panel"
        style = {
            display = Display.Flex
            flexDirection = FlexDirection.Column
            gap = 4.px
            padding = 6.px
            backgroundColor = 0xFF1F2A36.toInt()
            color = 0xFFE8EEF5.toInt()
        }
    }) {
        text("Card title")
    }
}
```

### DSS stylesheet rules

```dss { .css .copy .select }
:root {
  --accent: #4F8CC9;
}

.panel {
  border-width: 1px;
  border-color: #2B3E51;
}

.panel:hover {
  color: var(--accent);
}
```

Current DSS selector support is intentionally limited and explicit:

- type selectors: `button`, `text`, `div`, ...
- class selectors: `.panel`
- id selectors: `#settingsButton`
- universal selector: `*`
- pseudo states: `:hover`, `:active`, `:focus`, `:disabled`, `:open`
- combinators: descendant (` `), child (`>`), adjacent sibling (`+`), general sibling (`~`)
- `:root` for root variables (and root declarations)

## Value grammar in DSGL

### Length units

Supported units: `px`, `rem`, `em`, `vw`, `vh`, `%`.

- non-zero lengths must include a unit
- unitless `0` is accepted
- negative lengths are allowed only for selected properties (for example `margin`), rejected for
  size/border/gap/font-size

### Color literals

Supported color formats:

- `#RGB`
- `#RRGGBB`
- `#AARRGGBB`

### Other literals

- integers/floats for numeric properties (`z-index`, `flex-grow`, `opacity`, ...)
- booleans for `obfuscated` (`true/false`, plus parser aliases like `1/0`, `yes/no`)
- string paths for `background-image`
- `var(--name)` references to `:root` variables

## Supported Today

### Layout and box model

| Property                  | Accepted values                           | Notes                                 |
|---------------------------|-------------------------------------------|---------------------------------------|
| `display`                 | `block`, `inline`, `none`, `flex`, `grid` | Container flow mode                   |
| `width`, `height`         | CSS length                                | `auto` is not accepted as DSS literal |
| `min-width`, `min-height` | CSS length or `auto`                      | `auto` means unset                    |
| `max-width`, `max-height` | CSS length or `auto`                      | `auto` means unset                    |
| `margin`                  | 1-4 CSS lengths                           | Negative values allowed               |
| `padding`                 | 1-4 CSS lengths                           | Non-negative only                     |
| `gap`                     | CSS length                                | Non-negative                          |
| `align`                   | `start`, `center`, `end`                  | Used by stack/block alignment paths   |

Example:

```kotlin { .kotlin .copy .select }
style = {
    width = 50.percent
    minHeight = 40.px
    margin(4.px, 8.px)
    padding = 6.px
    gap = 3.px
    align = StyleAlign.CENTER
}
```

### Positioning and overflow

| Property                         | Accepted values                                     | Notes                                        |
|----------------------------------|-----------------------------------------------------|----------------------------------------------|
| `position`                       | `static`, `relative`, `absolute`, `fixed`, `sticky` | Position mode contract is runtime-tested     |
| `left`, `top`, `right`, `bottom` | CSS length or `auto`                                | Precedence: left over right, top over bottom |
| `z-index`                        | unitless integer                                    | Negative values allowed                      |
| `overflow`                       | `visible`, `hidden`, `scroll`, `auto` (1-2 values)  | Shorthand writes axis values                 |
| `overflow-x`, `overflow-y`       | `visible`, `hidden`, `scroll`, `auto`               | Axis-specific control                        |

Example:

```dss { .css .copy .select }
.panel {
  position: sticky;
  top: 0px;
  z-index: 100;
  overflow-x: visible;
  overflow-y: auto;
}
```

### Flex and grid properties

| Property           | Accepted values                                                           | Notes                           |
|--------------------|---------------------------------------------------------------------------|---------------------------------|
| `flex-direction`   | `row`, `column`                                                           |                                 |
| `justify-content`  | `start`, `center`, `end`, `space-between`, `space-around`, `space-evenly` |                                 |
| `align-items`      | `start`, `center`, `end`, `stretch`                                       | Used in flex and grid placement |
| `justify-items`    | `start`, `center`, `end`, `stretch`                                       | Used by grid placement path     |
| `flex-grow`        | float >= 0                                                                |                                 |
| `flex-shrink`      | float >= 0                                                                |                                 |
| `flex-basis`       | CSS length or `auto`                                                      |                                 |
| `grid-columns`     | int >= 1                                                                  | Numeric column count            |
| `grid-rows`        | int >= 1 or `auto`                                                        |                                 |
| `grid-auto-flow`   | `row`, `column`                                                           |                                 |
| `grid-column-span` | int >= 1                                                                  |                                 |
| `grid-row-span`    | int >= 1                                                                  |                                 |

Example:

```kotlin { .css .copy .select }
style = {
    display = Display.Grid
    gridColumns = 3
    gridAutoFlow = GridAutoFlow.Row
    gap = 4.px
    justifyItems = JustifyItems.Stretch
    alignItems = AlignItems.Start
}
```

### Color, border, image, transform, opacity

| Property           | Accepted values                | Notes                                             |
|--------------------|--------------------------------|---------------------------------------------------|
| `background-color` | hex color                      |                                                   |
| `background-image` | string path                    | Rendered as one stretched image to element bounds |
| `border-color`     | hex color                      |                                                   |
| `border-width`     | CSS length                     | Non-negative                                      |
| `border-radius`    | CSS length                     | Parsed/applied, but see partial section           |
| `color`            | hex color                      | Inherited                                         |
| `transform`        | `none` or function list        | Functions: `translate`, `scale`, `rotate`         |
| `transform-origin` | two components (`0..1` or `%`) |                                                   |
| `opacity`          | float (clamped to 0..1)        |                                                   |

Example:

```dss { .css .copy .select }
.preview {
  background-color: #223042;
  border-width: 1px;
  border-color: #36506A;
  transform: translate(6, -2) scale(1.05) rotate(3deg);
  transform-origin: 50% 50%;
  opacity: 0.92;
}
```

Inline DSL supports both property and function forms for complex values:

```kotlin { .kotlin .copy .select }
style = {
    border(StyleBorder(width = 1.px, color = 0xFF36506A.toInt()))
    border {
        width = 1.px
        color = 0xFF36506A.toInt()
    }

    transformOrigin(TransformOrigin(0.5f, 0.5f))
    transformOrigin {
        x = 0.5f
        y = 0.5f
    }

    margin {
        vertical(2.px)
        horizontal(4.px)
    }
    padding {
        all(1.px)
        left = 6.px
    }
    lineHeight {
        length(18.px)
    }
    overflow {
        x = Overflow.Hidden
        y = Overflow.Scroll
    }
    inset {
        top = 0.px
        left = 8.px
    }
}
```

### Text and font properties

| Property          | Accepted values                                                 | Notes                         |
|-------------------|-----------------------------------------------------------------|-------------------------------|
| `font-id`         | string                                                          | Inherited                     |
| `font-size`       | CSS length                                                      | Inherited; resolved to px     |
| `line-height`     | `normal` or CSS length                                          | Inherited                     |
| `font-weight`     | `normal`, `bold`                                                | Inherited                     |
| `font-style`      | `normal`, `italic`                                              | Inherited                     |
| `text-decoration` | `none`, `underline`, `strikethrough`, `underline-strikethrough` |                               |
| `obfuscated`      | boolean                                                         |                               |
| `text-wrap`       | `wrap`, `nowrap`                                                |                               |
| `text-formatting` | `none`, `minecraft`                                             | Enables MC formatting parsing |

Example:

```kotlin { .kotlin .copy .select }
style = {
    fontId = "minecraft"
    fontSize = 14.px
    lineHeight = LineHeightValue.Length(1.3.em)
    fontWeight = FontWeight.Bold
    textDecoration = TextDecoration.Underline
    textFormatting = TextFormatting.Minecraft
}
```

## Partially Supported / DSGL-specific Semantics

- `border-radius` is part of computed style and inspector data, but border/background rendering is still rectangular in
  current render command paths.
- `background-image` supports only a direct texture path and stretched draw to element bounds (no repeat/position/size
  controls).
- `overflow` shorthand with two different axis values stores axis values (`overflow-x`, `overflow-y`) and runtime
  behaviour follows those axes; the single `overflow` computed field is not a full CSS-equivalent state in that mixed
  case.
- `align` is not a global alignment model; it is used by container alignment paths where applicable (not a full CSS
  `align-self`/`justify-self` equivalent).
- Flex/grid behaviour is intentionally narrower than browser CSS (see [Layout model](layout-model.md) for exact runtime
  rules).

## Unsupported / Not Documented as Supported

These are not part of current DSGL style contract:

- extra `display` keywords such as `inline-block`, `contents`
- extra `position` keywords such as `inherit`, `unset`
- CSS functions like `calc()`, `min()`, `max()`, `clamp()`
- non-hex color syntaxes (`rgb()`, `rgba()`, named colors)
- advanced grid syntax (`fr`, `minmax`, template strings)
- CSS shorthands beyond the implemented ones (`margin`, `padding`, `overflow`)
- selector features like attribute selectors, pseudo-elements, media queries
- variable declarations outside `:root`

## Notes for maintainers

DSGL style semantics are SemVer-stable only where runtime/tests currently enforce behaviour. If you add a property or
expand semantics, update parser + style engine + runtime use + tests together.
