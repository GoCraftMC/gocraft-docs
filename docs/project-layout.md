# Project layout

A Go mod is laid out like a Fabric mod, with `src/main` for shared code and `src/client` for code that
only runs on the client:

```
mymod/
├── build.toml                   build configuration
├── go.mod, go.sum               Go dependencies
├── go.jar, gow, gow.bat         the build tool and its launchers
└── src/
    ├── main/
    │   ├── go/                  Go packages for both sides
    │   └── resources/
    │       ├── gomod.json       the manifest
    │       ├── assets/mymod/    textures, models, language files
    │       └── data/mymod/      recipes, loot tables, tags
    ├── client/
    │   ├── go/                  Go packages for the client
    │   └── resources/
    └── generated/go/            typed bindings, if you use them
```

## Packages, not package main

You never write `package main`. go.jar generates it at build time and imports every package under
`src/main/go` and `src/client/go`, and each one registers its entrypoints from `init`:

```go
func init() {
	gocraft.OnInitializeClient(setUpClient)
}
```

To link only some packages, list them under `entrypoints` in [gomod.json](gomod-json.md).

## The client split

Everything compiles into a single native library, so client packages are present on a dedicated server
too. They are safe because they only touch client classes from code that runs on the client:
`OnInitializeClient` handlers never run on a server, and Java members resolve on first use.

## Inside a .gomod

A `.gomod` is a zip file:

```
gomod.json
assets/, data/
natives/linux-amd64/mod.so
natives/windows-amd64/mod.dll
natives/darwin-arm64/mod.dylib
```
