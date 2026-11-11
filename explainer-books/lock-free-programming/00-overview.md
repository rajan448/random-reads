# Lock-Free Programming in Java — Explainer Book

> **You'll come away knowing:** How CAS, Java's atomic classes, and VarHandle support shared-state updates, and why ABA can break them.
> **For:** A beginner who can read basic Java · **Total reading time:** ~11 minutes

## Why this matters

Two threads can execute correct-looking code and still overwrite each other's work. A thread is an independently scheduled execution path. Shared updates need rules about indivisibility, visibility, and progress.

Lock-free programming lets competing threads advance without depending on a lock owner. It is useful for understanding concurrent libraries and reviewing small shared-state algorithms.

Think of editing a shared document. You read a revision, propose a change, and accept it only if your expected revision still matches. Compare-and-swap (CAS) supplies the conditional update; your algorithm supplies the rules around it.

## The 60-second version

- **Atomic** means an operation appears indivisible. Several atomic calls do not automatically form one atomic operation.
- **Compare-and-swap (CAS)** updates a value only if it matches an expectation. Java's `compareAndSet` reports success with a boolean.
- **Atomic classes** package common operations on individual values. **VarHandle** applies operations to existing storage with explicit ordering modes.
- **Lock-free** promises system-wide progress, assuming suitable primitives and continued thread execution. **Wait-free** additionally bounds each operation's own steps.
- **ABA** means a value changes and returns, hiding history from an equality check.

Java's [atomic package](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/atomic/package-summary.html) supplies the single-variable toolkit. [Herlihy's paper](https://cs.brown.edu/people/mph/Herlihy91/p124-herlihy.pdf) explains the progress distinction.

## Key terms

| Term | Plain-English definition |
| --- | --- |
| Lock / spin lock | Exclusive ownership / waiting repeatedly for ownership to become available. |
| Starvation | One thread repeatedly loses and cannot finish. |
| Visibility / memory ordering | Observable writes / constraints on how accesses can be observed. |
| `volatile` | Java modifier with visibility and ordering rules; it does not make increments atomic. |
| Retry loop | Read, compute, attempt CAS, and repeat after failure. |
| Linearization point | Instant an operation appears to take effect. |
| Invariant | Condition every operation must preserve. |
| Immutable state | Data that cannot change after construction. |
| Coherent access | Consistent ordering of accesses to one variable. |
| Release/acquire / publication | Matching ordering modes / making initialized data available through a handoff. |
| Spurious failure | A weak CAS fails even though its expected value matches. |
| Stack / node | Last-in-first-out collection / an element containing data and a link. |
| Stamp | Version value checked together with a reference. |
| Contention | Multiple threads competing for shared state. |

## Chapters

1. [01 — CAS and atomic classes](./01-cas-and-atomic-classes.md) — Trace retries and choose a suitable atomic API.
2. [02 — VarHandle and memory ordering](./02-varhandle-and-memory-ordering.md) — Access fields and understand a release/acquire handoff.
3. [03 — ABA and practical choices](./03-aba-and-practical-choices.md) — Follow a broken stack and reject a stale update using stamps.

## Scope and assumptions

Read this overview, then chapters 1–3. Examples assume basic Java syntax; prior concurrency knowledge is unnecessary.

As of November 2026, this guide uses Java SE 25 documentation as its reference baseline. Check contracts against your deployed JDK. It covers on-heap Java examples, excluding native-memory reclamation and complete production collections.

Algorithmic progress does not promise uninterrupted execution: scheduling and JVM garbage collection can pause threads. Use the examples to understand correctness, then prefer established library implementations for production structures.
