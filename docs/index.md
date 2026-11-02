# GoCraft

GoCraft lets you write a complete Minecraft mod in Go. Your mod compiles to native code with the full
Go runtime, so goroutines, channels and the whole standard library work, and it loads into the game
like any other mod: it has entrypoints, appears in mod lists, ships assets and data packs, and can
inject into game code.

```go
package mod

import (
	"github.com/GoCraftMC/gocraft-sdk/command"
	"github.com/GoCraftMC/gocraft-sdk/gocraft"
)

func init() {
	gocraft.OnInitialize(func() {
		command.Register(command.Literal("hello").Executes(func(c *command.Context) int {
			c.Reply("Hello from Go, " + c.SenderName() + "!")
			return 1
		}))
	})
}
```

Requires Minecraft 26.3 or newer, Java 25, Fabric Loader 0.20 and Fabric API.

## Where to go next

- [Getting started](getting-started.md): build and run your first mod.
- [Project layout](project-layout.md): what goes where.
- [gomod.json](gomod-json.md): the mod manifest.
- [go.jar](go-jar.md): the build tool.
- [The SDK](sdk.md): items, blocks, commands and events.
- [Reaching Minecraft](jvm.md): calling anything in the game, and overriding it.
- [Hooks](hooks.md): injecting into game code.
- [Threads and safety](threads.md): the rules that keep the game running.
- [Architecture](architecture.md): how it works.
- [Status](status.md): what is tested, and what is not done yet.
