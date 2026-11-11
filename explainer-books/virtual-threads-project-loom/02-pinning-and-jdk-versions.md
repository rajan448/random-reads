# 2. Pinning, contention, and JDK versions

> **TL;DR:** Pinning prevents unmounting; monitor-related pinning changed in JDK 24, while native interactions and ordinary resource contention still require attention.

## What pinning means

A virtual thread is **pinned** when it cannot detach from its carrier. If it then waits, the carrier cannot execute another virtual thread during that wait.

One short pinned interval may have little impact. Frequent, long waits can exhaust available carriers and leave runnable virtual threads waiting for execution. Pinning primarily threatens scalability; it does not automatically make application results incorrect.

Distinguish it from **lock contention**, multiple threads competing for the same protected resource. Threads can wait for a lock without pinning their carriers, yet the protected work remains serialized.

## Give the version before the rule

As of November 2026, distinguish these documented behaviors:

| Runtime or path | Pinning implication |
| --- | --- |
| JDK 21–23: blocking inside `synchronized` | The virtual thread cannot unmount because it holds an intrinsic monitor. |
| JDK 24+: ordinary Java synchronization | JEP 491 removes monitor-related pinning, including monitor acquisition and `Object.wait()`. |
| Native or foreign-code interaction | Native frames can still prevent unmounting; inspect the actual call path. |

An **intrinsic monitor** is the lock and waiting mechanism used by Java's `synchronized` and `Object.wait()`.

The [JDK 21 guide](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html) documents the earlier limitation. The [Quarkus runtime reference](https://quarkus.io/guides/virtual-threads/) explains the JDK 24 change and remaining native-to-Java callback cases.

**JNI**, the Java Native Interface, and the **Foreign Function and Memory API** let Java interact with native code. A native call that re-enters Java and then parks is an important remaining pinning example. A long native call can also occupy a carrier directly, whether or not a parking-related diagnostic event appears.

These are the main application-facing cases, not an exhaustive catalog of every VM path. Reproduce unusual library behavior on your exact JDK build.

## The same code can have different bottlenecks

This method excerpt deliberately holds a shared lock while waiting:

```java
private static final Object gate = new Object();

static void slowOperation() throws InterruptedException {
    synchronized (gate) {
        Thread.sleep(200); // Represents a long wait.
    }
}
```

On JDK 21–23, the sleeping virtual thread retains its carrier. On JDK 24+, ordinary synchronization no longer prevents it from unmounting.

However, ten calls sharing `gate` still take roughly ten sequential waits. `sleep` does not release the monitor. Removing pinning does not let multiple owners enter the same synchronized block.

Think of one bathroom key: freeing the people waiting in line does not create another bathroom.

On older JDKs, a **ReentrantLock**, an explicit lock supporting virtual-thread-friendly waiting, can address identified monitor-pinning hotspots. On newer JDKs, do not replace every `synchronized` block solely because of outdated advice. Reduce unnecessarily long lock ownership in either case.

## Diagnose the mechanism, not just the symptom

**JFR**, Java Flight Recorder, records runtime events. Capture `jdk.VirtualThreadPinned` and inspect stacks and duration thresholds. Use **jcmd**, the JVM diagnostic command tool, for a virtual-thread-aware dump:

```text
jcmd <pid> Thread.dump_to_file -format=json threads.json
jfr print --events jdk.VirtualThreadPinned recording.jfr
```

The second command assumes you have captured a JFR recording. An empty event list is not proof of healthy carrier availability. Check recording settings, CPU load, native blocking, and downstream queues. [Diagnostic reference](https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html).

`-Djdk.tracePinnedThreads=full` is an older JDK 21–23 diagnostic; it was removed in JDK 24. [Version-specific tooling note](https://quarkus.io/guides/virtual-threads/).

## Key takeaways

- State the JDK version when discussing pinning.
- Separate carrier retention from shared-lock serialization.
- Native integration still warrants investigation.
- Use recordings and thread dumps alongside application metrics.

**Next: [Design I/O-bound services and compare reactive stacks](./03-microservices-and-reactive-stacks.md)**
