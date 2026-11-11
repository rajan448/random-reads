# 3. Startup costs and debugging

> **TL;DR:** Measure startup stages and first-request behavior before changing bean creation or connection settings.

## What happens before useful work

Startup resembles opening a restaurant: unlock the building, prepare equipment, check supplies, then serve customers.
Each framework feature adds preparation; your initialization code and external systems add their own delays.

The **JVM**, Java Virtual Machine, runs Java applications. A simplified startup map is:

```text
JVM and class loading
  → configuration and condition checks
  → bean construction, dependency wiring, and proxy creation
  → database setup and web-server preparation
  → application runners and readiness
```

Actual stages interleave according to dependencies. **Application runners** are callbacks for startup work.
**Readiness** means your instance can handle its intended traffic; an open network port alone does not prove it.
[Spring application lifecycle](https://docs.spring.io/spring-boot/reference/features/spring-application.html).

Possible database costs include authentication, pool initialization, and **migrations**, controlled changes to database tables.
An **ORM**, object-relational mapper, maps objects to database tables; Hibernate is one example.
Its mapping setup can also contribute. These are candidates to measure, not a universal ranking.
[Boot database integration](https://docs.spring.io/spring-boot/reference/data/sql.html).

## Distinguish three clocks

| Clock | What you measure | What it reveals |
| --- | --- | --- |
| Process to readiness | Launch until representative work can succeed | Deployment and scaling delay. |
| First request | First representative request after readiness | Deferred initialization or empty caches. |
| Warm requests | Requests after repeated execution | Ongoing service cost. |

**JIT**, just-in-time compilation, turns frequently executed Java code into optimized machine code during execution.
Warmup is the period while execution and caches settle. Do not assume readiness means peak performance. [Oracle's HotSpot compiler explanation](https://www.oracle.com/java/technologies/whitepaper.html).

For a controlled experiment, keep hardware, dependencies, database state, and configuration comparable.
Repeat measurements and report their variation. Class-loading work and network delays can differ between launches.

## Collect evidence

As of November 2026, you can install Boot's startup recorder before calling `run`:

```java
// Inside main; MyApplication is your boot application class.
var app = new SpringApplication(MyApplication.class);
app.setApplicationStartup(new BufferingApplicationStartup(2048));
app.run(args);
```

Import `SpringApplication` from `org.springframework.boot` and the recorder from
`org.springframework.boot.context.metrics.buffering`.
The recorder stores startup steps; 2048 is its capacity, not a timeout.
[Startup tracking](https://docs.spring.io/spring-boot/reference/features/spring-application.html#features.spring-application.startup-tracking).

**Actuator**, Boot's operational monitoring module, can expose those steps through its `startup` endpoint when configured.
Its `conditions` endpoint explains conditional configuration choices. Alternatively, launch with `--debug` for the conditions report.
[Actuator endpoints](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html), [auto-configuration diagnostics](https://docs.spring.io/spring-boot/reference/using/auto-configuration.html).

## Changes with tradeoffs

As of November 2026, `spring.main.lazy-initialization=true` defers eligible bean creation until needed.
**Lazy initialization** can shorten startup while increasing first-request time and delaying configuration failures.
Exercise important request paths before declaring an instance ready. [Lazy initialization](https://docs.spring.io/spring-boot/reference/features/spring-application.html#features.spring-application.lazy-initialization).

**AOT**, ahead-of-time processing, prepares application infrastructure during the build.
GraalVM native images compile applications into platform-specific executables. They can reduce startup and memory costs,
but require handling dynamic features such as reflection, runtime inspection of classes.
They also constrain configuration decisions that determine which beans exist. [Native images and AOT](https://docs.spring.io/spring-boot/reference/packaging/native-image/introducing-graalvm-native-images.html).

## A practical diagnosis

Suppose startup steps show a six-second bean constructor calling a remote service.
That is a hypothetical diagnosis: changing proxy type cannot remove the network wait.
Move optional work out of startup, or make required work explicit in readiness.

For runtime pool waits, inspect borrowed connections, waiting callers, slow queries, and transaction duration together.
With three instances and a maximum of twenty connections each, budget up to sixty connections for that pool alone.

To check a transaction boundary, call the injected service and deliberately fail its second write.
Verify the first write disappears after rollback. Avoid a surrounding test transaction that could mask a missing service boundary.

## Key takeaways

- Separate readiness, first-request latency, and warmed performance.
- Optimize the measured cause; remeasure the same workload.
- Deferred work still needs time and resources later.

**Go deeper:** Trace `TransactionInterceptor`, the transaction boundary handler, and `DataSourceTransactionManager`, the JDBC transaction coordinator.
Use the linked Spring and HikariCP documentation to check behavior for your application's versions.
