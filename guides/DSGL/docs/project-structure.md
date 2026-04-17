# Project Structure

## Top-level modules

- `core/`
  DSGL runtime and public DSL that is not Minecraft-version-specific.

- `mc-forge-1-7-10/`
  Minecraft 1.7.10 screen host and renderer adapter integration on top of `core`.

- `mc-forge-1-7-10-demo/`
  Demo mod and showcase window/sections used as real usage examples and manual playground.

- `dsgl-hot-reload-agent/`
  Native Rust JVMTI agent submodule for hot-reload signaling. Built with Cargo, not with Gradle module tasks.

## `core/` structure

- `src/main/kotlin/org/dreamfinity/dsgl/core/Dsl.kt`
  Public DSL surface (`ui`, `UiScope`, element/component builders, style DSL access).

- `src/main/kotlin/org/dreamfinity/dsgl/core/DsglWindow.kt`
  Window contract and hook render-session lifecycle bridge for hosts.

- `src/main/kotlin/org/dreamfinity/dsgl/core/DomTree.kt`
  Retained DOM tree: reconcile, layout, paint command generation, ref commit.

- `src/main/kotlin/org/dreamfinity/dsgl/core/hooks/`
  Hook runtime and hook APIs (`useState`, `useEffect`, `useMemo`, etc.).

- `src/main/kotlin/org/dreamfinity/dsgl/core/style/`
  DSS parsing, stylesheet loading, style registry/model, computed style application.

- `src/main/kotlin/org/dreamfinity/dsgl/core/dom/`
  DOM node implementations, layout models, reconcile helpers, overflow/positioning behaviour.

- `src/main/kotlin/org/dreamfinity/dsgl/core/overlay/`
  Overlay layering/contracts and host-layer plumbing.

- `src/main/kotlin/org/dreamfinity/dsgl/core/components/`, `select/`, `contextmenu/`, `colorpicker/`, `dnd/`
  Higher-level public helpers and advanced interaction systems.

- `src/test/kotlin/...`
  Main regression coverage (hooks, layout, style, overlay/input, helper systems).

## `mc-forge-1-7-10/` structure

- `src/main/kotlin/org/dreamfinity/dsgl/mc1710/DsglScreenHost.kt`
  Main runtime host path for lifecycle, rebuild, input routing, and layer composition.

- `src/main/kotlin/org/dreamfinity/dsgl/mc1710/Mc1710UiAdapter.kt`
  Executes `RenderCommand` output against MC/LWJGL 1.7.10 APIs.

- `src/main/kotlin/org/dreamfinity/dsgl/mc1710/text/`
  MSDF runtime text rendering integration.

- `src/main/kotlin/org/dreamfinity/dsgl/mc1710/scissorsHelper/`
  Scissor/clipping helper code for backend rendering.

## `mc-forge-1-7-10-demo/` structure

- `src/main/kotlin/.../demo/ShowcaseWindow.kt`
  Main demo window.

- `src/main/kotlin/.../demo/sections/`
  Focused examples by feature area (layout, hooks, stylesheets, DnD, modal/select/context menu, etc.).

- `build.gradle.kts` + `gradle.properties`
  Demo run configuration, including optional hot-reload agent wiring.

## Build and release structure

- `settings.gradle.kts`
  Includes the Gradle multi-module build (`:core`, `:mc-forge-1-7-10`, `:mc-forge-1-7-10-demo`).

- `build.gradle.kts` (root)
  Shared tasks (font atlas generation/compression, module bump tasks, convenience aliases).

- `buildSrc/src/main/kotlin/*.conventions.gradle.kts`
  Internal Gradle convention plugins used by modules.

- `core/gradle.properties`, `mc-forge-1-7-10/gradle.properties`, `mc-forge-1-7-10-demo/gradle.properties`
  Module-level version/build flags and run properties.

## Docs and site

- `docs/`
  MkDocs content pages.

- `mkdocs.yml`
  Site nav and MkDocs configuration.

- `theme/`
  MkDocs Material overrides.

## Assets/tooling directories

- `fonts/`
  Source font assets.

- `precompiled_fonts*` directories
  Generated/prebuilt font artefacts consumed during resource packaging.

## Where should I change X?

- DSL/API behaviour: start in `core/.../Dsl.kt` and relevant `core/hooks` or component package.
- Runtime frame behaviour: start in `mc-forge-1-7-10/.../DsglScreenHost.kt` and `core/DomTree.kt`.
- Style parsing/application: start in `core/style/`.
- Overlay routing/ordering: start in `core/overlay/` plus host integration in `DsglScreenHost`.
- Demo regressions/examples: `mc-forge-1-7-10-demo/src/main/kotlin/.../demo/sections`.
