# 2. VarHandle and memory ordering

> **TL;DR:** VarHandle gives atomic operations and explicit ordering choices for existing fields and array elements.

## A handle to existing storage

An `AtomicInteger` owns its integer. A **VarHandle** describes how to access a variable stored elsewhere. An instance-field handle also needs the target object on every call.

Think of a handle as a control panel for a field. You choose ordinary reads, ordered reads, or conditional updates. The handle is not a lock. See the [VarHandle specification](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/VarHandle.html).

```java
import java.lang.invoke.MethodHandles;
import java.lang.invoke.VarHandle;

class FieldCounter {
    private volatile int value;
    private static final VarHandle VALUE;

    static {
        try {
            VALUE = MethodHandles.lookup()
                .findVarHandle(FieldCounter.class, "value", int.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    int increment() {
        for (;;) {
            int observed = (int) VALUE.getVolatile(this);
            if (VALUE.compareAndSet(this, observed, observed + 1)) {
                return observed + 1;
            }
        }
    }
}
```

`MethodHandles.lookup()` supplies access rights from the calling class. `findVarHandle` identifies the declaring class, field name, and field type. See the [Lookup API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#findVarHandle(java.lang.Class,java.lang.String,java.lang.Class)). Cache the handle in a `static final` field. Here, `this` selects the counter instance.

This has the same retry structure as an atomic wrapper. Prefer wrappers for straightforward application code; handles help when you need operations on existing storage. See [Doug Lea's VarHandle guide](https://gee.cs.oswego.edu/dl/html/j9mm.html).

## Atomicity and ordering are separate

**Memory ordering** constrains how threads can observe accesses relative to other accesses. An indivisible update alone does not explain visibility of surrounding data.

| Mode | Methods | What you can rely on |
| --- | --- | --- |
| Plain | `get`, `set` | No cross-thread ordering |
| Opaque | `getOpaque`, `setOpaque` | Coherent ordering for that variable; no publication of other data |
| Acquire/release | `getAcquire`, `setRelease` | Prior writes become visible through a matching release/acquire handoff |
| Volatile | `getVolatile`, `setVolatile` | Acquire/release properties plus a total order among volatile operations |

**Coherent** means accesses to that variable follow a consistent per-variable order. Plain `long` and `double` accesses also have weaker atomicity guarantees. See the [access-mode definitions](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/VarHandle.html).

## Publish data with a handoff

**Publication** makes initialized data available to another thread. Imagine packing a parcel before setting its “ready” sign.

For a one-shot handoff, assume one producer and one consumer share an already accessible mailbox. Its fields begin as `int payload = 0` and `boolean ready = false`. `READY` is a VarHandle for `ready`.

```java
// Producer: run once; do not overwrite the payload later.
mailbox.payload = 42;
READY.setRelease(mailbox, true);

// Consumer: run on another thread.
if ((boolean) READY.getAcquire(mailbox)) {
    int received = mailbox.payload; // Must be 42.
}
```

These are separate thread bodies, not a reusable queue. If the acquire reads the producer's `true`, it also sees the earlier payload write. If it reads `false`, this consumer attempt does nothing. Reusing the mailbox requires another coordination protocol. See [Lea's ordering guide](https://gee.cs.oswego.edu/dl/html/j9mm.html).

## Read the operation's exact contract

Strong `compareAndSet` uses volatile read semantics and, on success, volatile write semantics. Weak CAS may fail **spuriously**: without a mismatching value. Its suffix specifies ordering; `weakCompareAndSetPlain` supplies plain ordering.

The selected mode overrides a field's `volatile` declaration. Calling plain `get` on a volatile field still requests plain access. Avoid casually mixing modes. See [VarHandle](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/VarHandle.html).

## Key takeaways

- VarHandle accesses storage; atomic wrappers contain storage.
- Pair release publication with an acquire that observes it.
- Match ordering strength to a specific correctness argument.
- Check weak CAS and access-mode contracts explicitly.

**Next: [The ABA problem and practical choices](./03-aba-and-practical-choices.md)**
