# 2. Inlining and escape analysis

> **TL;DR:** Inlining exposes work across method boundaries; escape analysis can then help remove unnecessary objects and locks.

## Inlining opens the box

**Method inlining** places a called method's operations inside the caller's compiled representation.
Your source files remain unchanged.

Imagine unfolding a helper's instructions onto your own worksheet.
You can now spot duplicated work and unnecessary intermediate steps.

```java
static int twice(int x) { return x * 2; }
static int answer() { return twice(21); }
```

After inlining, the compiler can see `21 * 2` inside `answer`.
**Constant folding**, calculating constant expressions during compilation, can reduce this to `42`.
Inlining therefore unlocks benefits beyond avoiding a method call.

Inlining depends on method size, call frequency, receiver information, and compilation budgets.
Excessive inlining produces larger native code, which can hurt performance.
Both C1 and C2 inline; C2 generally pursues more extensive optimizations.

## Predictions preserve correctness

For an interface call, runtime profiles may show that one receiver class dominates.
The compiler can generate a specialized fast path and inline that class's implementation.
**Speculation** means optimizing under an assumption protected by checks or tracked dependencies.

An unexpected receiver may use a fallback or trigger **deoptimization**, which reconstructs interpreter execution from compiled state.
The JVM can gather new profiles and compile again.
Your program must retain Java's behavior through these changes.
See [OpenJDK's performance techniques](https://wiki.openjdk.org/spaces/flyingpdf/pdfpageexport.action?pageId=11829405).

## Escape analysis follows references

**Escape analysis** determines whether references make an object accessible outside the region being analyzed or its thread.
The **heap** is the memory area used for ordinary Java object allocation.

```java
final class Pair {
    final int x, y;
    Pair(int x, int y) { this.x = x; this.y = y; }
}

static int score(int n) {
    Pair p = new Pair(n, n + 1);
    return p.x + p.y;
}
```

If C2 can see the constructor and all relevant uses, `p` need not exist as an allocated object.
**Scalar replacement** replaces an object's fields with individual values.
The resulting computation could resemble `n + (n + 1)`.
This is a possible optimization, not a guarantee about every compilation.

Inlining matters because it exposes constructors, getters, and helpers to the same analysis.
Even an object returned by a helper may disappear after that helper is inlined into its caller.

> 💡 **Allocation removal is not stack allocation.** In this HotSpot model, the object may disappear entirely. Remaining values can occupy registers or stack slots.

Escape analysis can also support **lock elimination**, removing synchronization when analysis proves the relevant locking unnecessary.
See [Oracle's JDK 25 escape-analysis documentation](https://docs.oracle.com/en/java/javase/25/vm/java-hotspot-virtual-machine-performance-enhancements.html).

## Why an allocation remains

Storing `p` in a shared static field makes it globally accessible.
Passing it to code the compiler cannot analyze may prevent elimination.
Non-escaping objects can also remain because scalar replacement has additional limits.

Classic C2 analysis is conservative across branches: an escape on one path can prevent removal on another.
The [OpenJDK escape-analysis report](https://cr.openjdk.org/~cslucas/escape-analysis/EscapeAnalysis.html) explains these limitations.

## Key takeaways

- Inlining makes more optimization opportunities visible together.
- Speculative optimizations include mechanisms that preserve correctness.
- Escape analysis enables transformations; it does not guarantee allocation removal.
- A `new` expression does not prove that optimized execution allocates an object.

**Next: [Warm-up in containers](./03-container-warm-up-and-observation.md)**
