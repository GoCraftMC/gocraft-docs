# Architecture

## The shape of it

```
 .gomod ──▶ GoLanguageAdapter ──▶ Fabric Loader's mod list
                  │
                  ▼
             native library  ◀── Foreign Function & Memory API ──▶ Minecraft
             (Go runtime + your mod)
```

## Why native code

The options were compiling Go to WebAssembly, or compiling it to native code and calling it from Java.

WebAssembly gives one portable, sandboxed file with no C compiler needed. But Go's WebAssembly target has no
parallelism and no sockets, so goroutines stop being concurrent and `net/http` stops working. That is not
"a full mod in Go". Native code keeps all of Go, at the cost of building per platform, which go.jar handles
by cross-compiling.

Each mod is built with `-buildmode=c-shared` and so carries its own Go runtime. Whether many runtimes could
share a JVM was the main risk, so it was tested first: 24 runtimes in one JVM ran about 40 million upcalls
from goroutines while another thread forced garbage collection, null pointer exceptions and stack overflows,
without a crash. A round trip from Java to Go to Java to Go took about 147 ns on the test machine.

## How a Go mod becomes a real mod

Fabric Loader instantiates language adapters in `finishModLoading()`, after its solver has run but before it
reads entrypoints, access wideners or mixin configs. `GoLanguageAdapter`'s constructor uses that moment to:

1. Scan `mods/gomods` and check dependencies, the ABI and the platform.
2. Unpack each mod into `.gocraft/runtime`: its native library, and a generated jar holding its resources, a
   `fabric.mod.json` converted from `gomod.json`, and any hook mixins.
3. Add those jars to Loader's mod list. The loop that is instantiating the adapter holds an iterator on the
   current list, so the field is replaced with a new list rather than the old one changed.

From there nothing is special. Fabric reads the entrypoints and mixins itself, Fabric API builds resource
packs from the mod's files, and mod lists see an ordinary mod.

This depends on Loader internals. A Loader change that breaks it stops the game with a message saying GoCraft
needs updating.

## The bridge

Every call crosses through a table of function pointers given to each library at boot. Java's side is
`bridge/Abi.java` and Go's is `jvm/bridge.h`, and they must match: `ABI_VERSION` guards this at load time.

- **Go to Java.** Objects are 64-bit handles into a table. Generations in the handle turn a stale reference
  into an exception rather than a wrong object. Methods are resolved once to an id and a `MethodHandle`.
- **Java to Go.** Go closures get callback ids. Java passes arguments as raw 64-bit slots on a per-thread
  native stack.
- **Lifetime.** Each Go wrapper has a `runtime.AddCleanup` that releases its handle when collected.
  References passed into a callback stay owned by Java unless Go takes them, so an unused argument costs no
  extra call.
- **Errors.** An exception escaping into native code would abort the JVM, so every upcall catches everything
  and returns a status. Java exceptions become Go panics, Go panics become Java exceptions, and an exception
  that passes through Go comes out as the original.
- **Pointers.** They cross as real C pointers, never integers, so cgo pins them. A nested callback can grow a
  goroutine's stack, which would invalidate a stack address Java still held.

## Generated classes

`jvm.Extend` writes a subclass with ASM whose overrides forward to Go, with a `super$name` twin for each.
Hooks are the same idea at load time: `@Inject` mixins generated into the mod's runtime jar.

## Repositories

| Repo | Contents |
| --- | --- |
| `gocraft` | The Fabric mod, go.jar and the integration test. |
| `gocraft-sdk` | The Go SDK. |
| `gocraft-example` | A tour of the SDK, as a mod. |
| `gostats` | A small complete mod with unit tests and an in-game self-test. |
| `gocraft-docs` | This manual. |
| `gocraft-webdocs` | The site generator for this manual. |
