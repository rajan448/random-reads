# 3. Bound context and concurrent work

> **TL;DR:** Scoped values bound context lifetime; structured concurrency bounds the lifetime of related child tasks.

## Context belongs to an operation

**Request context** is information such as a tenant identifier needed across several calls. A **scoped value** binds that information while a particular operation executes.

Unlike a mutable `ThreadLocal`, a per-thread storage mechanism, its binding ends automatically when the operation exits. This includes exceptional exits. Code holding the key can read the binding; keep the key private.

Nested operations can temporarily rebind the key, restoring the enclosing value afterward. The object itself is not frozen. Prefer immutable context records, and keep ordinary parameters for dependencies that should remain explicit. [ScopedValue behavior](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ScopedValue.html)

## Related tasks need a shared lifetime

**Structured concurrency** groups child tasks inside a scope owned by their parent operation. Think of a work crew that must leave together.

You **fork** to start a child task and **join** to wait for the group's outcome. Closing the scope waits for its child threads to terminate. Java's `try`-with-resources statement closes the scope automatically. [Structured concurrency guide](https://docs.oracle.com/en/java/javase/25/core/structured-concurrency.html)

The default uses **virtual threads**, lightweight threads scheduled by the Java runtime. They help scale blocking work; they do not make CPU calculations intrinsically faster. [Virtual thread guide](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html)

## Run a Java 25 preview example

Save as `ContextDemo.java`. The fake services use sleep to simulate waiting.

```java
import java.util.concurrent.StructuredTaskScope;

public class ContextDemo {
    record Context(String tenant) {}
    private static final ScopedValue<Context> CONTEXT =
            ScopedValue.newInstance();

    static String dashboard(boolean fail) throws InterruptedException {
        try (var scope = StructuredTaskScope.open()) {
            var profile = scope.fork(() -> {
                Thread.sleep(200);
                return "profile:" + CONTEXT.get().tenant();
            });
            var orders = scope.fork(() -> {
                Thread.sleep(20);
                if (fail) {
                    throw new IllegalStateException("orders unavailable");
                }
                return 3;
            });
            scope.join();
            return profile.get() + " / orders:" + orders.get();
        }
    }

    public static void main(String[] args) throws Exception {
        try {
            String result = ScopedValue.where(CONTEXT, new Context("acme"))
                    .call(() -> dashboard(args.length > 0));
            System.out.println(result);
        } finally {
            System.out.println("bound afterward: " + CONTEXT.isBound());
        }
    }
}
```

Use **JDK 25** for both commands:

```sh
javac --enable-preview --release 25 ContextDemo.java
java --enable-preview ContextDemo
java --enable-preview ContextDemo fail
```

The successful run prints `profile:acme / orders:3`, then `bound afterward: false`. The failure run prints the cleanup line and exits with an exception.

The scope captures the context binding when opened. Its children inherit that binding; arbitrary executor submissions do not automatically inherit it. Open and fork under the same bindings.

In Java 25, the default join policy cancels unfinished siblings when a child fails. `join()` throws `StructuredTaskScope.FailedException`, preserving the underlying cause. Other **joiners**, policies for combining task outcomes, support different requirements. [Java 25 scope API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/StructuredTaskScope.html)

## Cancellation is cooperative

Cancellation interrupts child threads. A task that ignores interruption can delay scope closure indefinitely. Preserve interruption and configure service timeouts. The scope also supports a timeout through its configuration; neither cancellation nor a timeout rolls back an external write.

## Match the API to your release

**As of November 2026**, scoped values are final in Java 25. Structured concurrency remains preview in released Java 27. Java 21 uses `new StructuredTaskScope.ShutdownOnFailure()` followed by `join().throwIfFailed()`; Java 25 uses factories and joiners. Java 27 changes exception handling again. Do not copy preview code across releases unchanged. [Java 21 guide](https://docs.oracle.com/en/java/javase/21/core/structured-concurrency.html), [Java 27 guide](https://docs.oracle.com/en/java/javase/27/core/structured-concurrency.html)

## Key takeaways

- Bind context around the whole operation, including its task scope.
- Read child results after a successful join.
- Handle failure, interruption, and timeouts deliberately.
- Pin preview examples to a matching compiler and runtime release.

**Go deeper:** Change the context, trigger failure, and inspect cleanup. Then explore custom joiners in the release-specific scope API.
