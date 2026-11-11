# Modern Java Language Features — Explainer Book

> **You'll come away knowing:** How records, sealed types, and patterns make models safer, and how scoped values and structured concurrency clarify operation lifetimes.
> **For:** Developers comfortable with basic Java · **Total reading time:** ~11 minutes

## Why this matters

Java lets you express more intent directly in your code. You can declare data shapes, limit alternatives, and ask the compiler to check their handling.

Concurrent services need another kind of clarity: which operation owns context and child tasks. Together, these features reduce repeated code and make failures easier to reason about. They still require validation, thoughtful design, and appropriate tests.

## The 60-second version

- **Records** declare data carriers and generate consistent component access and object methods.
- **Sealed types** restrict direct subtypes, making a model's alternatives explicit.
- **Pattern matching** combines checks with extraction; exhaustive switches detect missing alternatives during compilation.
- **Scoped values** provide context bindings whose lifetime follows an operation.
- **Structured concurrency** gives related child tasks a common owner, completion policy, and cleanup boundary.

## Know your Java release

**As of November 2026**, distinguish final features from preview APIs. **Final** means a standard feature; **preview** means an opt-in feature whose design may change between releases.

| Feature | Java 21 | Later status relevant here |
| --- | --- | --- |
| Records | Final since 16 | No preview flags needed |
| Sealed classes and interfaces | Final since 17 | No preview flags needed |
| Type patterns with `instanceof` | Final since 16 | No preview flags needed |
| Record patterns and pattern `switch` | Final in 21 | No preview flags needed |
| Scoped values | Preview | Final in 25 |
| Structured concurrency | First preview | Fifth preview in 25; seventh in 27 |

See the official [language feature history](https://docs.oracle.com/en/java/javase/21/language/java-language-changes-summary.html), [Java 21 preview API list](https://docs.oracle.com/en/java/javase/21/docs/api/preview-list.html), [Java 25 scoped values](https://docs.oracle.com/en/java/javase/25/core/scoped-values.html), and [Java 27 structured concurrency](https://docs.oracle.com/en/java/javase/27/core/structured-concurrency.html). Check your target release's documentation before upgrading.

## Key terms

| Term | Plain-English definition |
| --- | --- |
| Component / accessor | A record's named value / method that reads it |
| Invariant / compact constructor | A validity rule / concise record validation body |
| Shallow immutability | References stay fixed; referenced objects may still change |
| Sealed type / permits | Restricted inheritance / its allowed direct subtypes |
| Pattern / cast | Checked extraction rule / explicit type conversion |
| Flow scoping | A variable is available where its match is known to succeed |
| Guard / dominance | Extra match condition / an earlier case covers a later case |
| Exhaustive switch | A switch covering every required alternative |
| Request context / scoped value | Operation metadata / a key with a bounded binding |
| ThreadLocal / virtual thread | Per-thread storage / a lightweight runtime-managed thread |
| Scope / fork / join | A task lifetime boundary / start a child / await the group |
| Joiner / interruption | Outcome policy / a cooperative signal to stop or respond |

## Chapters

1. [01 — Records and sealed models](./01-records-and-sealed-models.md) — Validate data and model alternatives without misleading immutability assumptions.
2. [02 — Pattern matching](./02-pattern-matching.md) — Use type and record patterns, guards, null handling, and exhaustive switches.
3. [03 — Scoped values and structured concurrency](./03-scoped-values-and-structured-concurrency.md) — Run a context-aware example and examine failure and cleanup.

## Scope and assumptions

You know classes, interfaces, methods, collections, and exceptions. Read chapters in order. Each contains a complete executable example.

The first two examples target Java 21 without preview features. The concurrency example targets Java 25 with preview enabled. You must use that JDK release to compile and run it. Scoped values and structured concurrency are library APIs, despite often being grouped with modern language features.

This short book covers the requested features. It skips framework integration, every new Java feature, custom joiner implementations, and detailed performance tuning. The goal is practical fluency in safety and maintainability, rather than syntax memorization.
