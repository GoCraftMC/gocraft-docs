# Hooks

A hook is a mixin written in Go: it runs your function at a chosen point in a game method. You declare it
in `gomod.json`, because mixins must be known before the game's classes load:

```json
"hooks": [
  {
    "id": "motd",
    "target": "net.minecraft.server.MinecraftServer",
    "method": "getMotd",
    "desc": "()Ljava/lang/String;",
    "at": "RETURN"
  }
]
```

and bind it in Go:

```go
func init() {
	gocraft.Hook("motd", func(h *gocraft.HookContext) {
		h.SetReturn(h.ReturnValue().String() + " (powered by Go)")
	})
}
```

## Fields

| Field | Meaning |
| --- | --- |
| `id` | The name you bind in Go. |
| `target` | The class to inject into. |
| `method`, `desc` | The method's name and JVM descriptor. |
| `at` | `HEAD`, `RETURN` or `TAIL`. Defaults to `HEAD`. |
| `static` | `true` if the method is static. |
| `environment` | `*`, `client` or `server`. Use `client` for client classes. |

A bad target stops the game at startup with Mixin's error, as with any required mixin.

## HookContext

| Method | Meaning |
| --- | --- |
| `Self()` | The object the method ran on. Null for static methods. |
| `Arg(i)`, `ArgCount()` | The method's arguments. Primitives arrive boxed. |
| `Cancel()` | Leaves a void method early. |
| `SetReturn(v)` | Leaves the method, returning `v`. Primitives must be boxed. |
| `ReturnValue()` | The value about to be returned, at `RETURN`. |

## Cost

A hook boxes its arguments and crosses into Go on every call. That is fine on most methods, but be careful
on ones that run thousands of times a tick. Prefer a Fabric event where one exists.

For more precise injection than these three points, write the mixin in Java: a Fabric mod and a Go mod can
depend on each other like any two mods.
