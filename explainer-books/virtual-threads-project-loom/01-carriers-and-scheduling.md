# 1. How virtual threads use carrier threads

> **TL;DR:** Many virtual threads share platform threads over time; a virtual thread needs a carrier only while executing or when blocking prevents detachment.

## Two schedulers, two thread identities

A **platform thread** normally maps one-to-one to an operating-system thread. The operating system schedules it onto a processor.

A **virtual thread** is a Java `Thread` scheduled by the **JVM**, the Java Virtual Machine. A **carrier** is the platform thread currently executing that virtual thread.

The JVM **mounts** a virtual thread on a carrier to execute it. **Unmounting** detaches its execution state so the carrier can run another virtual thread. One virtual thread does not execute on multiple carriers simultaneously. [Scheduling explanation](https://docs.oracle.com/en/java/javase/24/core/virtual-threads.html).

Think of virtual threads as many resumable jobs and carriers as a smaller set of workers. A job waiting for a delivery need not reserve its worker.

## Follow a blocking socket read

For a supported blocking network operation:

1. Virtual thread A executes on carrier 1 and calls a socket read.
2. If the data is unavailable, the runtime arranges to resume A when the operation can progress.
3. A unmounts; carrier 1 runs virtual thread B.
4. When ready, A becomes runnable and waits for scheduling.
5. A resumes, possibly on carrier 2, and its read completes.

```text
carrier 1: [A executes] [B executes] [other work]
A:         running → waiting, unmounted → runnable → running
carrier 2:                              [A resumes]
```

The calling code still looks synchronous. **Blocking** describes the caller waiting for completion; it need not mean an operating-system thread remains blocked. Not every operating-system or library operation supports this detachment. [Virtual-thread behavior](https://docs.oracle.com/en/java/javase/26/core/virtual-threads.html).

## What the scheduler manages

In the JDK 25 reference implementation, the default scheduler uses a dedicated **ForkJoinPool**, a worker pool supporting distributed task queues. It uses FIFO scheduling mode, meaning first-in-first-out local scheduling, rather than the common pool used by many asynchronous APIs. This is an implementation detail, not a universal Java guarantee. [JDK 25 scheduler source](https://raw.githubusercontent.com/openjdk/jdk/jdk-25-ga/src/java.base/share/classes/java/lang/VirtualThread.java).

Its target **parallelism**, the number of carriers available for scheduling work, defaults to the JVM's available-processor count. Carrier count is not the number of active requests, and need not be permanently fixed. [Scheduler configuration](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Thread.html).

**Concurrency** means tasks have overlapping lifetimes. **Parallelism** means executing tasks at the same time. Thousands of waiting requests do not require thousands of simultaneously running processors.

Long CPU computations still occupy carriers. Do not assume the virtual-thread scheduler automatically time-slices every CPU loop. The operating system can time-slice carriers, but that differs from the JVM switching virtual threads on one carrier.

## A small Java 21+ example

```java
import java.util.concurrent.Executors;

class Demo {
    public static void main(String[] args) throws Exception {
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            var result = executor.submit(() -> {
                Thread.sleep(100); // Stand-in for waiting, not a benchmark.
                return Thread.currentThread().isVirtual();
            });
            System.out.println(result.get()); // true
        }
    }
}
```

An **executor** accepts tasks for execution. This factory creates a fresh virtual thread per submitted task, without pooling virtual threads. [Executor factory contract](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Executors.html).

`Thread.currentThread()` returns the virtual thread, not its carrier. Your logical thread identity survives carrier changes. Virtual threads are daemon threads, which do not keep the JVM alive alone; joining or awaiting tasks matters. [Thread contract](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Thread.html).

## Key takeaways

- The JVM schedules virtual threads; the operating system schedules carriers.
- Supported waits release carriers while preserving synchronous code.
- Resumption need not use the same carrier.
- More waiting tasks do not create additional CPU capacity.

**Next: [Pinning, contention, and JDK versions](./02-pinning-and-jdk-versions.md)**
