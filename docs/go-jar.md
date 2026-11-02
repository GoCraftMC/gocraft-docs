# go.jar

go.jar is GoCraft's build tool, a single dependency-free jar. If you know Gradle, the model is the same:
a graph of tasks, run in parallel, that skip work that is already done.

```sh
./gow build              # compile, package, verify
./gow tasks              # list tasks
./gow runC               # any unambiguous prefix works
```

## Tasks

| Task | What it does |
| --- | --- |
| `build` | Assembles and verifies the mod. |
| `assemble`, `gomod` | Builds the `.gomod` without verification. |
| `compileGo` | Compiles every target. `compileGo-<target>` does one. |
| `check`, `vet`, `test` | `go vet` and `go test`. |
| `fmt`, `tidy`, `dependencies` | `go fmt`, `go mod tidy`, `go list -m all`. |
| `runClient`, `runServer` | Starts the real game with your mod. |
| `prepareClient` | Downloads what `runClient` needs without starting it. |
| `genBindings` | Generates typed Go bindings from the game. |
| `clean` | Deletes `build/` and task state. |
| `wrapper` | Writes `go.jar`, `gow` and `gow.bat` into the project. |

## Flags

| Flag | Effect |
| --- | --- |
| `-t` | Rebuild whenever inputs change. |
| `--rerun` | Ignore up-to-date checks. |
| `--no-cache` | Skip the build cache. |
| `--dry-run` | Show what would run. |
| `--continue` | Keep going after a failure. |
| `--workers=N` | Limit parallel tasks. |
| `--profile` | Write a timing report to `build/reports/profile.html`. |
| `--stacktrace` | Show stack traces for failures. |
| `--accept-eula` | Agree to the Minecraft EULA for `runServer`. |

## build.toml

```toml
[project]
id = "mymod"
version = "1.0.0"

[minecraft]
version = "26.3"
loader = "0.20.1"
fabric_api = "0.165.0+26.3"
gocraft_jar = "libs/gocraft.jar"

[go]
version = "system"        # or "1.26.1" to have it downloaded and verified
targets = ["linux-amd64", "windows-amd64", "darwin-arm64"]
build_flags = []

[properties]              # extra ${tokens} for gomod.json
homepage = "https://example.com"

[bindings]
include = ["net.minecraft.world.entity.animal.**"]

[run]
jvm_args = ["-Xmx4G"]
extra_gomods = ["../otherlib/build/libs/otherlib-1.0.0.gomod"]
```

## Incremental builds and the cache

Every task hashes its inputs. If they and the task's outputs are unchanged, the task reports
`UP-TO-DATE`. Compilation is also kept in a build cache in `~/.gocraft/build-cache`, shared by all your
projects, so going back to a previous state reports `FROM-CACHE` instead of recompiling. Local `replace`
directories in `go.mod` count as inputs.

## Toolchains

`[go] version` picks the compiler. `"system"` uses the one on your `PATH`, and a version number is
downloaded and checked against Go's published checksum.

Each entry in `targets` becomes a native library in the `.gomod`, and anything other than your own
platform is cross-compiled with a [Zig](https://ziglang.org) that go.jar downloads, so one machine can
build for all six supported targets. When building for macOS this way, the pure Go DNS resolver and user
lookup are used because there is no macOS SDK to link against.

## Custom tasks

```toml
[tasks.docs]
description = "Generates the wiki"
exec = "go run ./tools/wiki"
inputs = ["src/main/go"]
outputs = ["build/wiki"]
depends_on = ["genMain"]
finalizes = ["build"]     # build now runs this too
```

With `outputs` declared, a custom task is skipped when nothing changed, like the built-in ones.

## Compared with Gradle

| Gradle | go.jar |
| --- | --- |
| Task graph, parallel execution | Yes |
| Up-to-date checks | Yes |
| Build cache | Yes, local |
| Continuous build (`-t`) | Yes |
| Wrapper, abbreviated task names | Yes |
| Toolchain provisioning | Yes, Go and Zig |
| Custom tasks | Declarative, in `build.toml` |
| Build profile report | Yes |
| `init` | Yes |
| Dependencies | Through Go modules |
| Scripting and plugins | Not yet |
| Multi-project builds | Not yet: use `extra_gomods` |
| Daemon | Not needed so far: startup is fast |
| Remote build cache | Not yet |
| Publishing | Not yet |
