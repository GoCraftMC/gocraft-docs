# Threads and safety

Your mod runs as native code in the same process as the game. That gives you full speed and the whole Go
ecosystem. It also means a few rules matter more than they do in Java.

## Game state belongs to its thread

Event handlers, commands and callbacks run on the thread that fired them, normally the server thread, and
may touch the game freely. Goroutines may not: Minecraft is not thread-safe. Hand results back instead:

```go
gocraft.Go(func() {
	result := expensiveWork()
	server.Execute(func() { // back on the server thread
		server.Broadcast(result)
	})
})
```

Calling Java from a goroutine is safe in itself, so thread-safe things like `java.lang.String` or your own
concurrent code are fine. The rule is about game state.

## Do not block the game thread

Anything slow in a handler stalls the tick. Do network calls and heavy computation in a goroutine.

## Panics

A panic in a handler, on a thread the game called you on, is recovered. It becomes a Java
`GoPanicException` carrying your Go stack trace, and the game treats it like any exception from a mod.

**A panic in a goroutine you started with plain `go` ends the entire process, game included.** Go gives
libraries no way to stop this. Start goroutines with `gocraft.Go`, which logs the panic instead:

```go
gocraft.Go(func() { doWork() })
```

The same applies to libraries that start their own goroutines. Fatal runtime errors, such as concurrent map
writes, cannot be recovered by anything and also end the process.

## os.Exit

`os.Exit`, `log.Fatal` and the like end the game. Use them as you would in a library, which is to say never.

## Cgo and signals

Go and the JVM cooperate on signal handling, and many Go runtimes can share the process (see
[Architecture](architecture.md)). If you call C code that installs signal handlers, you can break this.
