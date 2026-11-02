# The SDK

`github.com/GoCraftMC/gocraft-sdk` is what your mod imports.

| Package | Purpose |
| --- | --- |
| `gocraft` | Entrypoints, logging, config, hooks. |
| `registry` | New items and blocks. |
| `event` | Game events. |
| `command` | Commands. |
| `mc` | Common game types: players, worlds, positions, stacks. |
| `jvm` | Anything else in the game. See [Reaching Minecraft](jvm.md). |

## Entrypoints

```go
func init() {
	gocraft.OnPreLaunch(func() { /* before the game's classes load */ })
	gocraft.OnInitialize(func() { /* both sides: register content here */ })
	gocraft.OnInitializeClient(func() { /* physical client only */ })
	gocraft.OnInitializeServer(func() { /* dedicated server only */ })
}
```

Do not use the game from `init` itself, because the bridge is not ready until the mod has loaded. Calling
it too early panics with a message saying so.

## Logging

`gocraft.Log()` is a `*slog.Logger` that writes to the game log under your mod's name. The standard `log`
and `slog` packages are redirected there too, so libraries you import log in the right place.

```go
gocraft.Log().Info("generated structure", "x", x, "z", z)
```

## Config

```go
cfg := Config{Greeting: "Hello"} // defaults
err := gocraft.LoadConfig(&cfg)  // reads config/<modid>.json, creating it if missing
```

## Items and blocks

```go
wand := registry.RegisterItem("wand", registry.Item{
	MaxStack: 1,
	OnUse: func(ctx registry.UseItem) mc.InteractionResult {
		ctx.Player.Send("Zap!")
		return mc.Success()
	},
})

ore := registry.RegisterBlock("ore", registry.Block{
	Hardness: 3, Resistance: 3, Light: 7, RequiresTool: true,
	OnStep: func(ctx registry.StepOn) { /* an entity walked on it */ },
})

registry.AddToTab(registry.TabTools, wand)
registry.AddToTab(registry.TabBuildingBlocks, ore.Item)
```

A bare id uses your mod's namespace. Models, textures, names and recipes go in `src/main/resources`, as
in any mod.

## Events

```go
event.ServerTick(func(s mc.Server) { /* 20 times a second */ })
event.ServerStarted(func(s mc.Server) {})
event.ServerStopping(func(s mc.Server) {})
event.PlayerJoin(func(p mc.Player, s mc.Server) { p.Send("Welcome") })
event.PlayerLeave(func(p mc.Player, s mc.Server) {})
event.BlockBroken(func(l mc.Level, p mc.Player, pos mc.BlockPos, st mc.BlockState) {})
event.UseBlock(func(p mc.Player, l mc.Level, pos mc.BlockPos) mc.InteractionResult { return mc.Pass() })
event.UseItem(func(p mc.Player, l mc.Level) mc.InteractionResult { return mc.Pass() })
event.ClientTick(func(client jvm.Object) {}) // from OnInitializeClient only
```

For any other Fabric event, see [Listening to any event](jvm.md#listening-to-any-event).

## Commands

```go
command.Register(
	command.Literal("gopher").Then(
		command.Literal("say").Then(
			command.Arg("message", command.GreedyString()).Executes(func(c *command.Context) int {
				c.Server().Broadcast(c.String("message"))
				return 1
			}),
		),
	),
)
```

Arguments are `Word`, `QuotedString`, `GreedyString`, `Bool`, `Int`, `IntRange` and `Double`.
