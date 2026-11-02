# Reaching Minecraft

The SDK's friendly packages cover the common cases. For everything else there are two routes: typed
bindings generated from the game, and the `jvm` package underneath.

## Typed bindings

List the classes you want in `build.toml`:

```toml
[bindings]
include = [
  "net.minecraft.world.item.Items",            # one class and its nested classes
  "net.minecraft.world.entity.animal.*",       # one package
  "net.minecraft.world.level.block.**",        # a package and its subpackages
]
```

`./gow genBindings` reads the actual game jar and writes `src/generated/go/mcgen/bindings.go`:

```go
import "example.com/mymod/src/generated/go/mcgen"

diamonds := mcgen.ItemStack_New_ItemLike_Int(mcgen.Items_DIAMOND(), 5)
count := diamonds.GetCount()
```

- Instance methods keep their names, capitalized: `getCount` becomes `GetCount`.
- Static methods and fields are prefixed with the class: `Items_DIAMOND`.
- Overloads after the first get their parameter types as a suffix.
- Classes outside your list appear as `jvm.Object`.

The list exists because Java packages import each other in cycles that Go forbids, so everything goes in
one Go package, and generating the whole game would make every build pay for thousands of unused types.

## The jvm package

Anything in the game is reachable by name. Minecraft 26.1 and newer ship unobfuscated, so these are the
same names you see in your IDE, and they include private members: there is no need for access wideners.

```go
var (
	getHealth = jvm.M("net.minecraft.world.entity.LivingEntity", "getHealth", "()F")
	maxPlayers = jvm.SF("some.Class", "MAX_PLAYERS", "I")
)

health := getHealth.Call(player).Float()
```

`M`, `SM`, `Ctor`, `F` and `SF` declare instance methods, static methods, constructors, instance fields
and static fields. They resolve on first use, so a package variable can name a client-only class safely.

Descriptors are JVM descriptors, such as `(ILjava/lang/String;)V`. `javap -s` prints them.

## Java exceptions

Every Java call can throw, so a thrown exception arrives as a panic with a `*jvm.Throwable`. Catch it where
you can handle it:

```go
err := jvm.Try(func() {
	parseInt.CallStatic("not a number")
})
```

If you do not catch it, it passes through your Go code and back into Java as the original exception.

## Implementing Java interfaces

```go
comparator := jvm.Func("java.util.Comparator", "compare(Ljava/lang/Object;Ljava/lang/Object;)I",
	func(a *jvm.Args) any { return strings.Compare(a.String(0), a.String(1)) })

task := jvm.Runnable(func() { fmt.Println("ran") })
```

`jvm.Proxy` implements several methods at once. Default methods you do not list keep their behaviour.

## Overriding Java classes

`jvm.Extend` defines a real Java subclass. In each callback, argument 0 is `this`:

```go
var class *jvm.Class
class = jvm.Extend(jvm.FindClass("net.minecraft.world.item.Item"), "my_item", jvm.Impl{
	"getDefaultInstance()Lnet/minecraft/world/item/ItemStack;": func(a *jvm.Args) any {
		stack := class.CallSuper(a.Object(0), "getDefaultInstance", "()Lnet/minecraft/world/item/ItemStack;")
		// change the stack...
		return stack
	},
})
item := class.New("(Lnet/minecraft/world/item/Item$Properties;)V", properties)
```

Constructors mirror the superclass. `CallSuper` is Java's `super.method()`.

## Listening to any event

Fabric events take an interface implementation, which is one `jvm.Func` away:

```go
const class = "net.fabricmc.fabric.api.event.lifecycle.v1.ServerLifecycleEvents"
event := jvm.FindClass(class).StaticField("AFTER_SAVE", "Lnet/fabricmc/fabric/api/event/Event;").GetStatic()
listener := jvm.Func(class+"$AfterSave", "onAfterSave(Lnet/minecraft/server/MinecraftServer;ZZZ)V",
	func(a *jvm.Args) any { return nil })
jvm.FindClass("net.fabricmc.fabric.api.event.Event").Method("register", "(Ljava/lang/Object;)V").Call(event, listener)
```

## Object lifetime

A `jvm.Object` keeps its Java object alive until Go's garbage collector drops the wrapper, so you can store
one anywhere. Arguments in a callback may be kept too, with `a.Object(i)`; the `Args` itself is valid only
during the call.
