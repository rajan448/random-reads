# 3. The ABA problem and practical choices

> **TL;DR:** CAS checks the present value; it cannot tell you whether that value changed and returned.

## The same value can hide a different history

The **ABA problem** occurs when a value changes from A to B and back to A, invalidating an earlier assumption.

Imagine checking a meeting room's “available” sign. Someone can enter and leave before you check again. The sign matches, but the history differs.

ABA is harmful when your proposed update depends on that history or on related state. An increment usually needs only the current number; a linked structure can depend on previously read links.

## Follow a stack failure

A **stack** removes the most recently added item first. Suppose its head, the top node, points through `A → B → C`. A node stores a value and a link to its successor.

Consider a flawed stack that permits reusing the same mutable nodes. Each removal reads the head and successor, then CASes the head to that successor.

| Step | Thread 1 | Thread 2 | Head chain |
| --- | --- | --- | --- |
| 1 | Saves head A and successor B; pauses | | A → B → C |
| 2 | | Removes A | B → C |
| 3 | | Removes B | C |
| 4 | | Reuses A, sets its link to C, pushes A | A → C |
| 5 | CASes head from A to saved B; succeeds | | B → C |

This constructed execution restores B even though thread 2 already removed it. The head comparison is correct; the assumption about A's successor is stale.

Java garbage collection reclaims unreachable objects. It does not prevent this logical reuse of a still-reachable node. Allocating fresh nodes and never reinserting removed nodes avoids this particular identity cycle.

## Compare a reference and a version

`AtomicStampedReference<T>` stores a reference with an integer **stamp**, a version number your algorithm maintains. Its CAS compares both together. Read the pair using `get(holder)`; separate reference and stamp reads can observe different updates. See [AtomicStampedReference](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/atomic/AtomicStampedReference.html).

```java
import java.util.concurrent.atomic.AtomicStampedReference;

class AbaDemo {
    public static void main(String[] args) {
        Object a = new Object();
        Object b = new Object();
        var head = new AtomicStampedReference<>(a, 0);

        int[] holder = new int[1];
        Object saved = head.get(holder);
        int version = holder[0];

        head.compareAndSet(a, b, 0, 1);
        head.compareAndSet(b, a, 1, 2);

        boolean staleUpdate =
            head.compareAndSet(saved, b, version, version + 1);
        System.out.println(staleUpdate); // false
    }
}
```

This sequential demonstration reproduces the decisive ordering deterministically. The reference returns to A, but version 2 differs from saved version 0.

Every relevant transition must advance the stamp. Integer stamps eventually wrap; they do not provide an unlimited history. A boolean mark also repeats, so `AtomicMarkableReference` is not a universal ABA remedy.

## Choose an approach you can justify

| Situation | Useful starting point |
| --- | --- |
| One independent value | Atomic class |
| Existing field needing atomic operations | VarHandle |
| Several related values | Lock, or CAS a fresh immutable state object |
| History-sensitive reference reuse | Versioned state or a node-lifetime protocol |

An **immutable state object** never changes after construction. Replacing it with a fresh object on every update can make identity changes explicit. Do not reintroduce old state objects.

**Contention** means threads compete for the same state. CAS retries consume processor time; lock-free does not automatically mean faster. Benchmark representative workloads before replacing a clear lock-based design.

For further study, try OpenJDK's [jcstress concurrency testing tool](https://github.com/openjdk/jcstress). Tests help expose executions; they cannot prove every possible interleaving safe.

## Key takeaways

- CAS checks equality now, not unchanged history.
- ABA can occur in Java when identities or values are reused.
- Stamps work only with a consistent update protocol.
- Explain your invariant, visibility rule, and progress argument separately.

**Go deeper:** Recreate the stack timeline, then explain why a changing stamp rejects the stale update.
