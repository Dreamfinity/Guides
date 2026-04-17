# Status and Limitations

## Stable

These areas are used across core + demo and have broad test coverage:

- window + screen host model (`DsglWindow`, `DsglScreenHost`)
- retained DOM tree build/reconcile/render flow
- base DSL elements (`div`, `text`, `button`, `input`, `textarea`, `toggle`, `select`, `img`)
- window-owned state (`state(...)`) and core hooks (`useState`, `useReducer`, `useMemo`, `useCallback`, `useEffect`,
  `useContext`, `useRef`)
- stylesheet loading and style application pipeline (`.dss` + inline style)
- primary layout modes in current model (`block`, `inline`, `flex`, `grid`, `none`) and positioning (`static`,
  `relative`, `absolute`, `fixed`, `sticky`)
- overlay layer ordering contract used by host/runtime (
  `application root -> application overlay -> system overlay -> debug`)

## Experimental

- drag-and-drop/sortable ecosystem (`useDraggable`, `useDroppable`, `useSortable`, monitor callbacks)
- color picker flows (especially popup/eyedropper interactions across overlay ownership)
- hot-reload integration path (stable enough for local development, but dependent on external agent/build workflow)

These areas are public and tested, but still advanced and more likely to evolve than baseline DSL/state/style/layout
surfaces.

## Internal

Do not treat these as extension contracts:

- `org.dreamfinity.dsgl.core.*.internal.*` packages (modal internals, inspector internals, overlay internals,
  color-picker internals)
- runtime plumbing classes such as `HotReloadBridge`, low-level overlay hosts, internal engine state holders
- system/debug overlay internals and inspector implementation nodes

If you rely on these directly, expect breakage across normal development changes.

## Known gaps / stubs / TODOs

- `ElementHandle.scrollIntoView()` is currently a TODO/no-op
- style/layout semantics are intentionally partial compared to browser CSS; only documented DSGL semantics should be
  treated as supported
- `border-radius` is parsed and represented in computed style, but border/background rendering remains rectangular in
  current paint paths
- `background-image` support is intentionally narrow (single texture path, stretched draw).
- advanced CSS grammar/features (media queries, `calc()`, attribute selectors, rich grid syntax, etc.) are not
  implemented as public DSGL behaviour
- Minecraft font rendering might look bad on different platforms and resolutions - known issue, will be fixed

## Not yet public / not ready to document as public extension model

- direct runtime mounting/reconcile internals as a custom-component authoring mechanism
- low-level JVMTI/Rust hot-reload internals as a stable contract
- generalized overlay framework internals behind modal/select/context-menu convenience DSLs

Use documented DSL composition patterns as the supported extension model:

- [Quickstart](quickstart.md)
- [Custom components](custom-components.md)
- [Hooks](hooks.md)
- [Architecture overview](architecture.md)
