# Getting started

You need Java 25. go.jar can fetch the rest: set a version under `[go]` in `build.toml` and it downloads Go,
and if you have no C compiler, which Go's native library mode requires, it downloads Zig and uses that.

## Create a project

Build `go.jar` and the mod jar from the [gocraft repo](https://github.com/GoCraftMC/gocraft), as there are no
release downloads yet. Then run `go.jar` in an empty directory:

```sh
mkdir mymod && cd mymod
java -jar /path/to/go.jar init mymod
```

This creates the project, plus `gow` and `gow.bat`, which you use from now on. Put the GoCraft mod jar
at `libs/gocraft.jar`, then fetch the SDK and build:

```sh
./gow tidy
./gow build
```

The result is `build/libs/mymod-0.1.0.gomod`.

## Run it

```sh
./gow runClient              # the game, with your mod, in run/client
./gow runServer --accept-eula   # a dedicated server in run/server
```

The server needs you to agree to the [Minecraft EULA](https://aka.ms/MinecraftEULA). `--accept-eula`
records your agreement in `run/server/eula.txt`, so only pass it after reading it.

Look for `Hello from Go!` in the log.

## Add something

Edit `src/main/go/mod/mod.go`:

```go
package mod

import (
	"github.com/GoCraftMC/gocraft-sdk/gocraft"
	"github.com/GoCraftMC/gocraft-sdk/mc"
	"github.com/GoCraftMC/gocraft-sdk/registry"
)

func init() {
	gocraft.OnInitialize(func() {
		registry.RegisterItem("ruby", registry.Item{
			OnUse: func(ctx registry.UseItem) mc.InteractionResult {
				ctx.Player.Send("It glitters.")
				return mc.Success()
			},
		})
	})
}
```

Give it a name in `src/main/resources/assets/mymod/lang/en_us.json`:

```json
{ "item.mymod.ruby": "Ruby" }
```

Run `./gow runClient` again and type `/give @s mymod:ruby`. Assets work exactly as in any other mod;
[gocraft-example](https://github.com/GoCraftMC/gocraft-example) has a complete item and block.

## Install a mod

Players put GoCraft and Fabric API in `mods/`, and `.gomod` files in `mods/gomods/`.
