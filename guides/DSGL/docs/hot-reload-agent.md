# Hot-Reload Agent Setup

This page covers practical usage of `dsgl-hot-reload-agent` with DSGL:

- how to run DSGL with a prebuilt native agent library
- how DSGL consumes the hot-swap signal
- what is intentionally internal

## What the agent does (and does not do)

The native agent detects JVM class redefinition and signals DSGL to rebuild by calling `HotReloadBridge.markHotSwap()`.

It does not:

- perform class redefinition itself
- patch bytecode
- replace DSGL runtime logic

All of this is handled by your IDE / plugins HotSwap functionality

DSGL-side reaction is in `DsglScreenHost`: it checks `HotReloadBridge.consumeHotSwap()` during rebuild checks.

## Platform/version assumptions

- DSGL host path here is Minecraft 1.7.10 (`mc-forge-1-7-10` / `DsglScreenHost`).
- Agent is a native JVMTI library (`cdylib`) and must match your OS.
- Expected default binary names:
    - Windows: `dsgl_hot_reload_agent.dll`
    - Linux: `libdsgl_hot_reload_agent.so`
    - macOS: `libdsgl_hot_reload_agent.dylib`

## Repository layout relevant to hot reload

- Agent project: `dsgl-hot-reload-agent/` (Cargo, not a Gradle module)
- DSGL bridge flag: `core/src/main/kotlin/org/dreamfinity/dsgl/core/HotReloadBridge.kt`
- Host rebuild integration: `mc-forge-1-7-10/src/main/kotlin/org/dreamfinity/dsgl/mc1710/DsglScreenHost.kt`
- Demo run wiring: `mc-forge-1-7-10-demo/build.gradle.kts`

## Installing

### Building from source code

Clone submodule `dsgl-hot-reload-agent` and build the release binary. See `dsgl-hot-reload-agent/README.md` for
details.

### Downloading prebuilt binaries

Download the latest release from [GitHub](https://github.com/Dreamfinity/dsgl-hot-reload-agent/releases) for your OS -
Windows, Linux and Intel-based macOS are supported. Archive already contains a properly pathed native library - so you
can extract it directly into your project.

## Running a demo client with hot reload enabled

### 1) Enable hot reload for demo run

In `mc-forge-1-7-10-demo/gradle.properties`:

```properties { .properties .copy .select }
hotReload=true
```

### 2) Optional: override binary file name

If your binary name does not match OS defaults, set:

```properties { .properties .copy .select }
hotReloadAgentLibraryName=<your-library-file-name>
```

`mc-forge-1-7-10-demo/build.gradle.kts` uses this to build `-agentpath:...` for `runClient`.

### 3) Start a demo client

```shell { .shell .copy .select }
.\gradlew :mc-forge-1-7-10-demo:runClient
```

When `hotReload=true`, run config adds:

`-agentpath:<repo>/dsgl-hot-reload-agent/target/release/<library>`

### 4) Verify rebuild signal is consumed

Expected behaviour in host logs/runtime:

- after a class is redefined, the host detects a hot-swap flag
- rebuild runs in hot-reload hook mode
- if hook signatures changed incompatibly, the host may remount/reset subtree and retry

One visible log from the host path is:

- `Hot swapped - re-building the DOM`


## How DSGL handles the signal

At runtime:

1. Agent sets `HotReloadBridge` atomic flag (`markHotSwap()`).
2. `DsglScreenHost.rebuildIfNeeded()` consumes flag.
3. Rebuild path switches to `HookRenderSessionMode.HotReload`.
4. Hook runtime may request remount/retry for incompatible hook signatures.

This behaviour is intentionally runtime-managed by host + hook runtime; apps should not call bridge internals directly.

## Using the agent outside `mc-forge-1-7-10-demo`

If you run another client configuration, you still need JVM `-agentpath` pointing to the native library.

The automatic `-agentpath` wiring in this repo is implemented in `:mc-forge-1-7-10-demo:runClient` task, not globally
for every run target.

Look demo example [buildscript](https://github.com/Dreamfinity/DSGL/blob/master/mc1710-demo/build.gradle.kts) to see
how to wire the agent with your Gradle run config.

## Troubleshooting

- `Unsupported operating system ... and 'hotReloadAgentLibraryName' is not set`
  Set `hotReloadAgentLibraryName` in `mc-forge-1-7-10-demo/gradle.properties`.

- No rebuilds after class redefined
  Check that:
    - agent is actually loaded (`-agentpath` present)
    - host path is `DsglScreenHost`
    - you are redefining classes in a way your tooling/JVM supports

- Repeated remount retries in hot-reload mode
  Usually means hook signature/order changed incompatibly for existing component paths.
  This is expected protective behaviour, not a stable migration API.

## Public API boundary note

`HotReloadBridge` exists for runtime integration between the native agent and host.
Treat it as an implementation detail, not as an application-level API.

Next: [Cookbook](cookbook.md)
