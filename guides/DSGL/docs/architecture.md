# Architecture

## Core model

- A window (`DsglWindow`) defines UI by returning a DOM tree in `render()`.
- A screen host drives lifecycle, input, rebuild scheduling, layout, and painting.
- The retained DOM tree is reconciled across rebuilds instead of recreated from scratch every frame.
- A per-window hook runtime tracks hook state/effects across rebuilds.

## Main execution path

In MC 1.7.10, the primary loop is in `DsglScreenHost`:

1. `initGui()` creates/attaches the window and host services.
2. Each frame (`drawScreen`), the host checks whether a rebuild is needed.
3. If needed, the host calls `window.render()` inside a hook render session.
4. The new tree is reconciled into the retained tree (`DomTree.reconcileWith(...)`).
5. Layout commit runs through `DomTree.render(...)`.
6. Paint commands are generated via `DomTree.paint(...)`.
7. The adapter (`Mc1710UiAdapter`) executes render commands.

The host can still paint previously committed commands if rebuild/layout/paint fails in a frame.

## Rebuild and reconcile

Rebuild triggers include:

- window invalidation (`state(...)`, hook state changes)
- viewport/size changes
- hot-reload signal (when enabled)

On rebuild, DSGL:

- builds a fresh tree from the window
- reconciles into the retained tree
- keeps compatible nodes
- cleans up detached subtrees/listeners

This is why stable keys matter for focus/state continuity.

## Style resolution in the runtime

Style data is resolved from:

- inline style DSL on elements/components
- DSS stylesheets loaded by `StylesheetManager`
- inspector overrides (application scope)

`StyleEngine` applies computed styles before layout and also during paint when style revisions or pseudo/selector state
require it.

Style scope is explicit:

- application/root/app-overlay trees use application scope
- system overlay uses system scope

## Layout and paint responsibilities

`DomTree.render(...)`:

- applies styles for current scope
- resolves layout style values
- performs layout
- commits ref bindings

`DomTree.paint(...)`:

- updates style/scroll/sticky visual state if needed
- rebuilds render-command chunks when dirty
- emits final render commands for the frame

So, layout is not a one-time setup step, it is part of the frame pipeline when invalidated.

## Hook runtime integration

The host controls hook session boundaries:

- `beginRenderBuild(...)`
- `endRenderBuild()`
- `commitRenderBuild()` on successful rebuild
- `discardRenderBuild()` on failed/aborted rebuild

This is the basis for DSGL effect semantics (commit-bound effects, discard-safe behaviour).

When hot-reload mode is active, hook signature mismatches can request subtree remount/reset instead of crashing the
entire rebuild.

## Overlay model

DSGL composes layers, not separate rendering universes:

- application root
- application overlay
- system overlay
- debug layer

Paint order and input priority are explicit in `OverlayLayerContracts`.

You usually treat overlays as host-managed runtime surfaces. Public helper APIs (modal/select/context-menu/color picker,
etc.) sit on top of this model; low-level overlay internals are not a stable app extension API.

## Where to inspect next

- Runtime host path: `mc-forge-1-7-10/src/main/kotlin/org/dreamfinity/dsgl/mc1710/DsglScreenHost.kt`
- Retained tree + reconcile/layout/paint: `core/src/main/kotlin/org/dreamfinity/dsgl/core/DomTree.kt`
- Style runtime: `core/src/main/kotlin/org/dreamfinity/dsgl/core/style/StyleEngine.kt`
- Stylesheet loading: `core/src/main/kotlin/org/dreamfinity/dsgl/core/style/StylesheetManager.kt`
- Hook lifecycle runtime: `core/src/main/kotlin/org/dreamfinity/dsgl/core/hooks/ComponentHookRuntime.kt`
