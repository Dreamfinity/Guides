# DSGL documentation

!!! warning "Alpha-release"

    Keep in mind that DSGL is currently in alpha-release. Public API might be changed, some bugs might be 
    present - so please report any issues you encountered to the [issue tracker](https://github.com/Dreamfinity/DSGL/issues) 

## Start here

- [Quickstart](quickstart.md)
- [Architecture overview](architecture.md)

## Documentation map

Core concepts:

- [State and reactivity](state-reactivity.md)
- [Custom components](custom-components.md)
- [Architecture overview](architecture.md)
- [Project structure](project-structure.md)
- [Hot-reload agent setup](hot-reload-agent.md)
- [Cookbook](cookbook.md)
- [Troubleshooting](troubleshooting.md)
- [Status and limitations](status-limitations.md)

Reference:

- [Elements overview](elements-overview.md)
- [Styling](styling.md)
- [Layout model](layout-model.md)
- [Hooks](hooks.md)

## Terminology used in these docs

- **window**: a `DsglWindow` implementation that builds the DOM tree
- **screen host**: a host implementation (for example `DsglScreenHost`) that drives lifecycle, input, and painting
- **DOM tree**: the retained tree of DSGL nodes (`DomTree` + `DOMNode` hierarchy)
- **hook runtime**: the per-window hook state/effect system managed by `ComponentHookRuntime`
- **overlay**: layered UI surfaces above the application root (application/system/debug layers)
- **stylesheet**: DSS files parsed and applied by the style engine
- **element**: a concrete node in the DOM tree (for example, text, button, input, container)
- **component**: a reusable UI composition built with the DSL
- **props**: a set of attributes passed to a component. Might extend ComponentProps or be a custom type

## Scope and status

- Documentation is implementation-grounded: if behaviour is partial, experimental, internal, or stubbed, it is
  documented as such
- Browser-CSS assumptions are intentionally avoided unless DSGL behaviour clearly matches
- This documentation tries to follow repository behaviour as much as possible
