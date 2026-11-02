# Status

GoCraft is at version 0.1, and its API may still change.

## Tested

`scripts/integration-test.sh` in the `gocraft` repo runs in a real Minecraft 26.3 server, using Fabric's
headless test mode, and currently passes all 26 checks on Linux x86-64. They cover entrypoints, strings,
primitives, exceptions in both directions, proxies, generated subclasses, private member access, concurrent
goroutines, handle release, registries, block placement, commands, hooks from two mods, typed bindings and data
pack loading.

Also seen working on Linux x86-64:

- The real client starts, the Go client entrypoint runs, and the mod's resources are in the resource reload
  with no errors. Nobody has yet looked at the item and block in a world.
- Launching without any special JVM flags.
- Downloading and checking a Go toolchain, cross-compiling with a downloaded Zig, incremental builds, the build
  cache, custom tasks and continuous builds.
- Clear startup errors for a missing dependency and a missing platform.

## Not yet tested

- **Windows and macOS at runtime.** go.jar builds libraries for Windows and macOS, on x86-64 and arm64, and
  Linux on arm64, but so far they have only been checked to be valid files. The repo's CI runs the integration
  test on all three systems, and has yet to run.
- **Joining a world as a player**, so `PlayerJoin`, `UseBlock`, `BlockBroken` and the like have not fired from real
  input.
- **Mod Menu.** Go mods are in Loader's mod list, which is what it reads, but it has not been tried.
- **A dedicated server outside test mode.** That needs the Minecraft EULA accepted.
- **Building with no C compiler installed.** Zig is used for cross-compiling, but the fallback for your own
  platform has not run on a machine that actually lacks a compiler.

## Not done yet

- Publishing to Modrinth and CurseForge, and to Maven.
- Plugins and scripting in go.jar, and multi-project builds.
- Per-instance Go state on `jvm.Extend` classes, which block entities will want.
- Friendly SDK packages for networking, rendering, screens, entities and data generation. All of these are already
  reachable through `jvm` and typed bindings.
- More precise hook injection points than `HEAD`, `RETURN` and `TAIL`.
