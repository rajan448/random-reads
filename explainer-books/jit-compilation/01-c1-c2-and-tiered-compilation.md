# 1. C1, C2, and tiered compilation

> **TL;DR:** C1 produces useful machine code quickly; C2 spends more effort optimizing code that runs frequently.

## From bytecode to machine instructions

The Java Virtual Machine, or **JVM**, executes Java **bytecode**, the portable instructions produced by `javac`.
This book focuses on **HotSpot**, the JVM implementation used by many OpenJDK distributions.

Its **interpreter** executes bytecode instructions without first compiling each method into machine code.
A **just-in-time compiler**, or **JIT**, translates selected methods while your application runs.
The generated **native code** contains instructions for your actual processor.

Think of an interpreter as reading a recipe during cooking.
A compiler prepares a specialized routine you can repeat faster.
Preparation costs time, so compiling every rarely used recipe would waste work.

## What the two compilers do

**C1**, historically the client compiler, prioritizes short compilation time.
It converts bytecode into an **intermediate representation**, an internal model that supports optimization.
It simplifies expressions, can inline small methods, then lowers this model toward machine instructions.
Finally, it assigns values to processor registers and emits native code.
See the [OpenJDK compiler architecture discussion](https://cr.openjdk.org/~thartmann/papers/2014-Code_Cache_Optimizations-thesis.pdf).

**C2**, historically the server compiler, prioritizes performance after compilation.
It builds a richer graph of operations and dependencies, then performs more extensive analysis and rewriting.
For example, it can optimize loops, remove redundant checks, and eliminate certain object allocations.
It eventually selects machine instructions, assigns registers, and emits code.
The [HotSpot compiler overview](https://cr.openjdk.org/~vlivanov/talks/2015_JIT_Overview.pdf) describes these optimization techniques.

C2 recompiles bytecode using runtime information; it does not optimize C1's emitted machine instructions.
Both compilers consume CPU and memory while your application runs.

## Five execution levels

**Tiered compilation** combines interpretation and compilation levels.
A **profile** records observed behavior, such as branch choices and receiver types at method calls.
A receiver is the object whose method you invoke.

| Level | Execution | Information collected |
| --- | --- | --- |
| 0 | Interpreter | Counters and runtime profiles |
| 1 | C1 | No profiling instrumentation |
| 2 | C1 | Method-call and loop-backedge counters |
| 3 | C1 | Counters plus fuller runtime profiles |
| 4 | C2 | Optimized using available profiles |

A **backedge** returns execution to an earlier point in a loop.
These counts help identify **hot code**: code executed often enough to justify compilation.

```text
Common route:       interpreter (0) → C1 with profiling (3) → C2 (4)
Possible detour:    interpreter (0) → C1 (2) → C1 (3) → C2 (4)
Trivial method:     may settle at C1 (1)
```

Compilation queues and collected profiles influence transitions.
Methods can skip levels; they do not climb a mandatory staircase.
There is no universal “10,000 calls means C2” rule.
See the version-pinned [JDK 25 compilation policy](https://github.com/openjdk/jdk/blob/jdk-25-ga/src/hotspot/share/compiler/compilationPolicy.hpp).

## One long loop

Suppose you call a method once, but its loop runs millions of times.
**On-stack replacement**, or **OSR**, lets execution enter compiled code within that running method.
You need not wait for the next method call.
This is distinct from compilation for ordinary method entry.
See the [OSR explanation, slides 32–33](https://cr.openjdk.org/~vlivanov/talks/2015_JIT_Overview.pdf).

## Key takeaways

- C1 saves compilation time; C2 invests in sustained execution speed.
- Profiles help the JVM choose worthwhile optimizations.
- Compilation levels describe individual methods, not a single application-wide state.
- OSR makes long-running loops eligible for compiled execution.

**Next: [Inlining and escape analysis](./02-inlining-and-escape-analysis.md)**
