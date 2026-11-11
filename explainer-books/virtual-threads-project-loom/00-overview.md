# Virtual Threads and Project Loom — Explainer Book

> **You'll come away knowing:** How virtual threads share carriers, when pinning matters, and how to design waiting-heavy services alongside reactive alternatives.
> **For:** Java developers preparing for design discussions or interviews · **Total reading time:** ~11 minutes

## Why this matters

Project Loom brings lightweight concurrency to OpenJDK. Virtual threads represent concurrent tasks using familiar Java thread semantics.

Waiting becomes cheaper; CPU, memory, locks, and downstream capacity remain finite. That boundary shapes your execution model and bottlenecks.

## The 60-second version

- The JVM mounts virtual threads on platform threads called carriers. Supported waits can unmount them and free those carriers.
- Pinning prevents unmounting. Blocking inside `synchronized` is a major issue on JDK 21–23; JDK 24 removes monitor-related pinning.
- Use a virtual thread per concurrent task and explicit limits for scarce dependencies. Do not pool virtual threads just to conserve them.
- Reactive stacks offer asynchronous composition and demand-aware streams. Virtual threads simplify blocking request code without supplying those stream semantics automatically.

## Key terms

| Term | Plain-English definition |
| --- | --- |
| Project Loom | OpenJDK's effort to improve lightweight concurrency. |
| JDK / JVM | Java Development Kit / Java Virtual Machine executing the program. |
| Platform thread | A Java thread normally backed by one operating-system thread. |
| Virtual thread | A lightweight Java thread scheduled by the runtime. |
| Carrier | The platform thread currently executing a virtual thread. |
| Mount / unmount | Attach execution to a carrier / detach resumable execution state. |
| Blocking | The caller waits for an operation to complete. |
| Concurrency / parallelism | Overlapping task lifetimes / simultaneous task execution. |
| ForkJoinPool / FIFO | A worker pool with distributed queues / first-in-first-out scheduling mode. |
| Pinning | A virtual thread cannot detach from its carrier. |
| Intrinsic monitor | Java's built-in synchronization and waiting mechanism. |
| Lock contention | Multiple threads compete for the same lock. |
| ReentrantLock | An explicit Java lock supporting flexible acquisition and waiting. |
| JNI / foreign-function API | Interfaces for calling native code from Java. |
| JFR / jcmd | Java Flight Recorder / JVM diagnostic command tool. |
| Executor / Future | A task execution service / a task-result and cancellation handle. |
| Daemon thread | A thread that does not keep the JVM alive alone. |
| I/O-bound / CPU-bound | Work dominated by external waits / computation. |
| Thread-per-request | One logical thread follows a request's execution. |
| Connection pool / semaphore | Limited reusable connections / a permit-based concurrency gate. |
| Admission control / bulkhead | Bound accepted work / isolate a dependency's concurrency. |
| Deadline / timeout | Overall completion cutoff / an operation's waiting limit. |
| Little's Law | Average in-flight work equals throughput multiplied by average time. |
| Reactive stack / event loop | Asynchronous operation composition / workers handling ready callbacks. |
| Backpressure | Consumer-driven control over production rate or demand. |
| ThreadLocal | A value associated with a logical Java thread. |
| Tail latency | Response time near the slow end of the distribution. |

## Chapters

1. [01 — How virtual threads use carrier threads](./01-carriers-and-scheduling.md) — Trace mounting, I/O waiting, resumption, and scheduling.
2. [02 — Pinning, contention, and JDK versions](./02-pinning-and-jdk-versions.md) — Separate old synchronization limitations from remaining blocking problems.
3. [03 — Design I/O-bound services and compare reactive stacks](./03-microservices-and-reactive-stacks.md) — Choose execution style and place concurrency limits deliberately.

Read the overview, then chapters 1–3; rehearse the final question aloud.

## Scope and assumptions

You know Java methods, exceptions, and threads. Examples use Java 21+ APIs; `Demo` is the standalone program.

As of November 2026, this guide distinguishes the JDK 21 baseline from JDK 24+ monitor behavior, with implementation examples pinned to JDK 25. Check your precise runtime and libraries for additional release-specific changes.

This covers common application behavior, excluding exhaustive VM paths and framework configuration. Structured concurrency manages related task lifetimes; virtual threads also work independently.
