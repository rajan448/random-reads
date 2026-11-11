# 3. Design I/O-bound services and compare reactive stacks

> **TL;DR:** Virtual threads make direct blocking code scalable for waiting-heavy requests, while reactive stacks retain stream-composition and demand-management capabilities.

## Move the limit to the resource you protect

An **I/O-bound** service spends much of its request time waiting for network or storage operations. Virtual threads let you keep a straightforward **thread-per-request** design: one logical thread follows a request's execution.

Use ordinary calls, loops, exceptions, and resource cleanup. Represent concurrent tasks with virtual threads rather than maintaining a pool of reusable virtual workers.

But cheap waiting does not create database connections. A **connection pool** still limits how many connections can be borrowed. A **semaphore** limits access through a count of permits. [Semaphore contract](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Semaphore.html).

For a remote service, use a shared permit limit and a bounded acquisition wait:

```text
try to acquire a permit before the request deadline
if unavailable: reject or return an appropriate fallback
try: perform the remote call with its own timeout
finally: release the permit
```

This is pseudocode. A connection pool already provides its own resource limit; do not add a redundant semaphore without another purpose.

**Admission control** limits accepted work. A **bulkhead** isolates one dependency's concurrency from others. Bound both active calls and waiting requests; thousands of cheap parked threads still retain request data in memory.

## Estimate concurrency before choosing limits

**Little's Law** relates average in-flight work to throughput and time in a stable system:

```text
average in-flight requests = completed requests/second × average seconds/request
```

At 2,000 requests/second and 0.2 seconds per request, expect about 400 requests in flight. This illustrative calculation does not size the database pool: requests may spend different fractions of time using it.

Virtual threads can help represent those waiting requests. They do not make an overloaded dependency complete them faster.

Use deadlines spanning the request and timeouts for individual operations. Cancelling a **Future**, a task-result handle, requests cancellation; it does not guarantee a remote action stops. Closing an executor waits for submitted tasks, so a timeout on `get` alone is insufficient. [Executor lifecycle](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/ExecutorService.html).

## Compare programming models, not slogans

A typical **reactive stack** composes asynchronous operations through events and stream operators. An **event loop** executes ready callbacks on a small worker set. Blocking that worker delays unrelated callbacks. [WebFlux concurrency model](https://docs.spring.io/spring-framework/reference/web/webflux/new-framework.html).

| Concern | Virtual-thread request handling | Typical reactive stack |
| --- | --- | --- |
| Waiting | Suspend the logical request thread. | Resume processing through completion signals. |
| Application style | Direct calls and ordinary control flow. | Compose operators and asynchronous results. |
| Dependency APIs | Suits blocking network or database APIs. | Benefits from non-blocking APIs throughout. |
| Flow control | Explicit request and dependency limits. | Stream demand plus explicit resource limits. |
| Streaming | Requires suitable streaming APIs and limits. | Rich stream-composition operators. |

**Backpressure** lets a consumer control production to avoid being overwhelmed. Virtual threads do not implement Reactive Streams demand signaling. Reactive backpressure also does not automatically impose every database or service limit. [Reactive Streams specification](https://www.reactive-streams.org/).

The models can coexist at deliberate boundaries. Reactor supports running blocking tasks on virtual threads through an appropriately configured scheduler. That does not make blocking calls safe on its event-loop workers. [Reactor scheduling](https://projectreactor.io/docs/core/release/reference/coreFeatures/schedulers.html).

## Change a service deliberately

Keep CPU-heavy work separately bounded. Audit **ThreadLocal** caches, values stored per logical thread: one expensive object per request can multiply memory use. Carrier changes do not move a virtual thread's ThreadLocal values to another request. [Adoption guidance](https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html).

Compare completed throughput, tail latency, CPU, memory, pinning, and dependency waits under realistic load. Do not promise a faster remote call or rewrite a working reactive system without measured benefit.

## Key takeaways

- Limit scarce resources rather than pooling virtual threads.
- Bound admitted work, waits, and task lifetimes.
- Choose request control flow and streaming composition independently.
- Benchmark the complete service and its dependencies.

**Where to go deeper:** Explain why a service with 10,000 virtual threads and 50 database connections can still be overloaded.
