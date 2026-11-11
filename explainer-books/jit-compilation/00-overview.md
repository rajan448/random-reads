# JIT Compilation — Explainer Book

> **You'll come away knowing:** How HotSpot combines C1 and C2, why inlining enables allocation removal, and how container limits affect warm-up.
> **For:** Java beginners and engineers with basic programming knowledge · **Total reading time:** ~11 minutes

## Why this matters

Your Java service can execute the same method differently as it runs.
Early requests may use interpreted code; later requests may benefit from compiled, specialized code.
Understanding this progression helps you explain performance changes, evaluate benchmarks, and plan container rollouts.

## The 60-second version

- A just-in-time compiler creates processor instructions while your application runs.
- C1 compiles quickly; C2 spends more effort improving frequently executed code.
- Tiered compilation combines them, using runtime observations to guide optimization.
- Inlining exposes method bodies together. Escape analysis can help eliminate object allocations and locks.
- Warm-up depends on executed paths and available resources. Waiting or probing one health endpoint is insufficient evidence.

## The central loop

```text
Execute bytecode → observe behavior → compile selected methods → run native code
                          ↑                                       │
                          └──── reprofile if assumptions fail ──────┘
```

Treat optimized code as a revisable plan.
The JVM can change that plan while preserving your program's Java behavior.
The [OpenJDK execution-policy presentation](https://openjdk.org/projects/leyden/slides/leyden-jvmls-2023-08-08.pdf) illustrates this execution cycle.

## Key terms

| Term | Plain-English definition |
| --- | --- |
| JVM / HotSpot | Java's execution engine / the implementation discussed here. |
| Bytecode / native code | Portable JVM instructions / instructions for the actual processor. |
| Interpreter / JIT | Execute bytecode directly / compile selected code during execution. |
| C1 / C2 | HotSpot compilers favoring quick compilation / more extensive optimization. |
| Intermediate representation | A compiler's internal model of program operations. |
| Tiered compilation | Combine interpretation and compilation levels. |
| Profile / hot code | Observed runtime behavior / frequently executed code. |
| Receiver / backedge | Object receiving a method call / a jump back within a loop. |
| Inlining | Incorporate called operations into the caller's compiled representation. |
| Constant folding | Calculate constant expressions during compilation. |
| Speculation / deoptimization | Optimize under protected assumptions / return compiled execution to the interpreter. |
| OSR | On-stack replacement: enter compiled code inside a running method. |
| Heap / escape analysis | Object-allocation memory / analysis of where object references can become accessible. |
| Scalar replacement | Replace an object with its individual field values. |
| Lock elimination | Remove synchronization proved unnecessary. |
| Warm-up / steady state | Develop profiles and compiled code / reach reasonably stable workload performance. |
| Container / cgroup | Packaged, isolated process / Linux mechanism controlling grouped processes' resources. |
| CPU quota / throttling | CPU-time budget / enforced execution pauses when that budget is exhausted. |
| Garbage collection | Reclaim heap memory occupied by unused objects. |
| Readiness | Deployment decision that an instance can serve traffic. |
| Code cache | Memory storing generated native code. |
| JDK / JMH | Java Development Kit / Java Microbenchmark Harness. |

## Chapters

1. [01 — C1, C2, and tiered compilation](./01-c1-c2-and-tiered-compilation.md) — Follow execution levels and long-loop compilation.
2. [02 — Inlining and escape analysis](./02-inlining-and-escape-analysis.md) — Trace a helper call and an allocation that may disappear.
3. [03 — Warm-up in containers](./03-container-warm-up-and-observation.md) — Inspect compilation and design a repeatable startup experiment.

## Scope and assumptions

Read this overview, then chapters 1–3.
You need basic familiarity with methods, objects, and loops; compiler experience is unnecessary.

As of November 2026, this guide uses JDK 25 documentation and its conventional HotSpot C1/C2 configuration as a concrete baseline.
It does not claim identical behavior across every JVM, vendor, or Java release.
Check current documentation for your deployed build before applying flags or relying on defaults.

Alternative compilers, ahead-of-time compilation, saved startup state, and detailed processor instruction analysis are outside this short book.
Examples explain possible optimizations; they promise neither exact compilation thresholds nor a fixed warm-up duration.
