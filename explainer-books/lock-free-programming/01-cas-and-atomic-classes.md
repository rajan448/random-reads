# 1. CAS and Java's atomic classes

> **TL;DR:** CAS makes a conditional update indivisible; a retry loop builds a larger operation around it.

## Progress without an owner

A **thread** is an independently scheduled path through your program. A **lock** gives one thread exclusive access to shared work.

Think of a shared whiteboard. With a lock, one person holds the marker. If they pause, everyone waits.

**Lock-free** algorithms guarantee system-wide progress while active threads keep executing, assuming nonblocking atomic primitives. Individual threads can suffer **starvation**: repeatedly losing without finishing. **Wait-free** algorithms additionally bound each operation's own steps. These are algorithmic guarantees, not deadlines in milliseconds. See [Herlihy's progress definitions](https://cs.brown.edu/people/mph/Herlihy91/p124-herlihy.pdf).

A spin lock repeatedly checks an ownership flag. It still depends on the owner releasing that flag. Using CAS to acquire it does not make the protected operation lock-free.

## Why volatile increment loses updates

**Atomic** means an operation appears indivisible to competing threads. **Visibility** concerns which writes another thread can observe.

Java's `volatile` modifier supplies visibility and ordering rules, but `count++` still reads, adds, then writes. If two threads read 7, both can write 8. You expected 9. See the [Java memory model](https://docs.oracle.com/javase/specs/jls/se25/html/jls-17.html#jls-17.4).

**Compare-and-swap (CAS)** conditionally replaces a value. Java calls its boolean-returning operation `compareAndSet(expected, replacement)`.

The comparison and update happen atomically. Success returns `true`; a mismatch leaves the value unchanged and returns `false`. Reference CAS uses object identity, `==`, rather than `.equals()`. See [AtomicReference](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/atomic/AtomicReference.html#compareAndSet(V,V)).

## A complete retry loop

```java
import java.util.concurrent.atomic.AtomicInteger;

class CasCounter {
    private final AtomicInteger count = new AtomicInteger();

    int increment() {
        for (;;) {
            int observed = count.get();
            int updated = observed + 1;
            if (count.compareAndSet(observed, updated)) {
                return updated;
            }
        }
    }
}
```

If another increment wins first, you reread and recompute. The successful CAS is the **linearization point**: the instant the whole increment appears to happen.

Each failed strong CAS in this algorithm indicates another update intervened. Other calls progress even if yours keeps losing. For production increments, use `incrementAndGet()`. Like ordinary `int` arithmetic, this example wraps on overflow.

## Choose the smallest useful API

| Need | Starting point |
| --- | --- |
| Integer counter or sequence | `AtomicInteger`, `AtomicLong` |
| Single flag | `AtomicBoolean` |
| Replace an object reference | `AtomicReference<T>` |
| Independently atomic array elements | `AtomicIntegerArray`, `AtomicLongArray`, `AtomicReferenceArray<T>` |

These are single-variable tools; array operations apply per element, not across the array. See the [atomic package](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/atomic/package-summary.html).

`getAndIncrement()` returns the old value; `incrementAndGet()` returns the new one. `compareAndExchange()` returns the observed value instead of a boolean. Functions passed to `updateAndGet()` must avoid side effects because retries can invoke them repeatedly. See [AtomicInteger](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html).

For heavily updated statistics, consider `LongAdder`. It spreads updates across internal counters. Its `sum()` is not an atomic snapshot during concurrent updates, so avoid it for exact admission decisions. See [LongAdder](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/atomic/LongAdder.html).

## Preserve the whole rule

An **invariant** is a condition your program must preserve. Two atomic balances do not make a transfer atomic. Another thread can observe only one balance updated.

Use a lock for coupled changes, or CAS one immutable object containing the complete state. **Immutable** means that object's state never changes after construction.

`AtomicReference` makes its reference atomic, not the referenced object's mutable fields.

## Key takeaways

- Volatile visibility does not make a compound update atomic.
- CAS retries must reread and recompute.
- One atomic variable does not protect unrelated mutable state.
- Lock-free progress permits individual starvation.

**Next: [VarHandle and memory ordering](./02-varhandle-and-memory-ordering.md)**
