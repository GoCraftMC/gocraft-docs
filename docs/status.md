# Status

GoCraft is at version 0.1, a [pre-release](https://github.com/GoCraftMC/gocraft/releases), and its API may still change.

## Tested

`scripts/integration-test.sh` in the `gocraft` repo runs in a real Minecraft 26.3 server, using Fabric's
headless test mode. CI runs its 32 checks on Linux, Windows and macOS. They cover entrypoints, strings,
primitives, exceptions in both directions, proxies, generated subclasses, private member access, concurrent
goroutines, the release of handles and callbacks, registries, block placement, commands, hooks from two mods,
typed bindings and data pack loading.

A Fabric `FakePlayer` drives the game's server-side interaction code, so `BlockBroken`, `UseItem`, `UseBlock`
and the `OnUse` overrides fire through the same code a real player's actions reach.

Also seen working on Linux x86-64:

- The real client starts, the Go client entrypoint runs, and the mod's resources are in the resource reload
  with no errors. Nobody has yet looked at the item and block in a world.
- Launching without any special JVM flags.
- Clear startup errors for a missing dependency and a missing platform.
- Downloading and checking a Go toolchain, cross-compiling with a downloaded Zig, incremental builds, the build
  cache, custom tasks and continuous builds.
- Typed bindings for several thousand classes, which compile and pass `go vet`.
- A new project built from the released `go.jar`, mod jar and SDK, which loads in a real server.

## Not yet tested

- **Cross-compiled libraries at runtime.** CI compiles on each system with its own compiler. The Windows and
  macOS libraries go.jar builds from another system have been checked only to be valid files, so test one before
  shipping it. The macOS ones link against hand-written stubs of system libraries.
- **Arm64 Linux and Windows**, which CI does not run.
- **The client on Windows and macOS.** CI only runs the server.
- **A real player joining a world**, so `PlayerJoin` and `PlayerLeave` have never fired.
- **Mod Menu.** Go mods are in Loader's mod list, which is what it reads, but it has not been tried.
- **A dedicated server outside test mode.** That needs the Minecraft EULA accepted.
- **Building with no C compiler installed.** Zig is used for cross-compiling, but the fallback for your own
  platform has not run on a machine that actually lacks a compiler.

## Not done yet

- Publishing to Modrinth and CurseForge, and to Maven. Releases are on GitHub only.
- Plugins and scripting in go.jar, and multi-project builds.
- Per-instance Go state on `jvm.Extend` classes, which block entities will want.
- Friendly SDK packages for networking, rendering, screens, entities and data generation. All of these are already
  reachable through `jvm` and typed bindings.
- More precise hook injection points than `HEAD`, `RETURN` and `TAIL`.
