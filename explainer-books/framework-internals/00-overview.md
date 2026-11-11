# Framework Internals — Explainer Book

> **You'll come away knowing:** How Spring creates proxies, places transaction boundaries, borrows database connections, and spends startup time.
> **For:** A Java developer new to framework internals · **Total reading time:** ~11 minutes

## Why this matters

Your method may look ordinary while its caller actually reaches a framework wrapper.
Understanding that extra layer helps you explain missing rollbacks, exhausted connection pools, and slow application launches.

Follow one service call from object creation through database cleanup, then use that model to diagnose performance.

## The 60-second version

- Spring manages application objects; Spring Boot conditionally supplies supporting configuration.
- Proxies intercept selected calls and add behavior such as transaction management.
- A transaction's boundary determines which database operations succeed or roll back together.
- HikariCP lends reusable database connections; transaction duration can determine how long you hold them.
- Startup prepares infrastructure. Deferring preparation can move its cost into your first request.

The chapters connect [Spring's proxy model](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html) with [Boot's database integration](https://docs.spring.io/spring-boot/reference/data/sql.html).

## Key terms

| Term | Plain-English definition |
| --- | --- |
| Bean / bean definition | Managed object / blueprint describing its creation. |
| Application context | Spring's container for managing beans. |
| Dependency injection | Supply an object's collaborators from outside it. |
| Auto-configuration / classpath | Conditional framework setup / available Java classes and libraries. |
| Component scanning / callback | Discover annotated classes / method invoked during a lifecycle stage. |
| Definition processor / object processor | Hook changing blueprints / hook processing actual objects. |
| Auto-proxy creator | Processor selecting beans for wrapping. |
| Proxy / target / interceptor | Stand-in object / wrapped object / behavior around a call. |
| AOP | Aspect-oriented programming: shared behavior applied across methods. |
| JDK / CGLIB proxy | Java Development Kit interface wrapper / generated subclass wrapper. |
| Self-invocation | One method calls another on the same target object. |
| Transaction / commit / rollback | Database work unit / accept changes / undo changes. |
| JDBC / DataSource | Java Database Connectivity API / source of database connections. |
| Transaction manager | Coordinator of transaction begin, completion, and cleanup. |
| Auto-commit | Commit each database statement individually. |
| Thread / thread-local storage | Executing stream of work / data associated with that thread. |
| Propagation / savepoint | Transaction participation rule / partial rollback marker. |
| Rollback-only | Transaction state prohibiting a successful commit. |
| Checked exception | Exception Java normally requires callers to catch or declare. |
| Connection pool / starter | Reusable connection supply / Boot dependency bundle. |
| JVM / JIT | Java Virtual Machine / compilation during execution. |
| Readiness / warmup | Ability to serve intended traffic / settling into warmed execution. |
| Application runner / migration | Startup callback / controlled database structure change. |
| ORM / Actuator | Object-relational mapper / Boot's operational monitoring module. |
| Lazy initialization / AOT | Create when needed / prepare ahead of time during builds. |
| Native image / reflection | Platform-specific executable / runtime inspection of classes. |

## Chapters

1. [01 — Bean creation and proxies](./01-beans-and-proxies.md) — Follow creation, wrapping, and intercepted calls.
2. [02 — Transactions and connection pools](./02-transactions-and-connection-pools.md) — Trace transaction ownership and connection borrowing.
3. [03 — Startup costs and debugging](./03-startup-costs-and-debugging.md) — Measure preparation costs and evaluate practical tradeoffs.

## Scope and assumptions

You know basic Java classes, methods, annotations, and exceptions. Read this overview, then chapters 1–3.
Examples use imperative, blocking JDBC access to one relational database, where work waits for database calls to finish.

Reactive systems, which compose asynchronous work, use different transaction-context mechanics and are outside this book.
Distributed transactions and detailed ORM behavior also need separate treatment.
Code fragments illustrate mechanisms rather than a complete runnable application.

As of November 2026, the linked documentation describes current behavior. Check your Spring Boot, Spring Framework,
and HikariCP versions for changes to defaults and configuration support.
