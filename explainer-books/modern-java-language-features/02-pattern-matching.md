# 2. Match and handle every alternative

> **TL;DR:** Patterns combine checking and extraction; exhaustive switches make missing cases a compilation problem.

## A pattern checks before exposing a value

A **type pattern** checks an object's type and gives you a correctly typed variable. It removes a separate cast, the explicit instruction to treat an object as another type.

In `value instanceof String text && !text.isBlank()`, Java evaluates the second condition only after a successful match. The variable `text` is available where the compiler can prove the match succeeded. This is **flow scoping**. Replacing `&&` with `||` does not provide the same guarantee. [Type patterns and variable scope](https://docs.oracle.com/en/java/javase/21/language/pattern-matching-instanceof.html)

Think of a pattern as checking a parcel label before opening the matching compartment.

## Record patterns unpack components

A **record pattern** checks a record's type and extracts its components through accessors. You can nest patterns to inspect records inside records.

For `record Box(Outcome value) {}`, the pattern `Box(Accepted(int cents))` matches a box containing an accepted result. It does not match a box whose value is null or rejected. A null value never matches a record pattern. [Record pattern rules](https://docs.oracle.com/en/java/javase/21/language/record-patterns.html)

Use nesting when it clarifies the condition. Several levels of extraction can make a simple rule harder to read.

## A complete Java 21 example

Save this standalone example as `PatternDemo.java`. No preview flags are needed.

```java
public class PatternDemo {
    sealed interface Outcome permits Accepted, Rejected {}
    record Accepted(int cents) implements Outcome {}
    record Rejected(String reason) implements Outcome {}

    static String describe(Outcome outcome) {
        return switch (outcome) {
            case null -> "Missing result";
            case Accepted(int cents) when cents >= 10_000 ->
                    "Review: " + cents;
            case Accepted(int cents) -> "Ship: " + cents;
            case Rejected(String reason) -> "Reject: " + reason;
        };
    }

    static String nonblank(Object value) {
        if (value instanceof String text && !text.isBlank()) {
            return text.strip();
        }
        return "unknown";
    }

    public static void main(String[] args) {
        System.out.println(describe(new Accepted(12_000)));
        System.out.println(describe(new Accepted(500)));
        System.out.println(describe(new Rejected("out of stock")));
        System.out.println(describe(null));
        System.out.println(nonblank("  ready  "));
    }
}
```

Run `javac --release 21 PatternDemo.java`, then `java PatternDemo`. You get review, shipping, rejection, missing-result, and trimmed-text outputs.

## Guards, ordering, and completeness

A **guard**, introduced with `when`, adds a condition after matching. Put the specific guarded case before its broader counterpart.

An earlier case **dominates** a later case when it already handles everything that later case could match. Java rejects such unreachable cases. Moving the unguarded `Accepted` case first demonstrates this error.

An **exhaustive switch** covers all required alternatives. Here the two record variants cover every non-null `Outcome`. The explicit null branch defines a separate policy; `default` alone would not handle null. [Pattern switch semantics](https://docs.oracle.com/en/java/javase/21/language/pattern-matching-switch.html)

Avoid adding a catch-all default merely to silence the compiler for a closed model. Try adding `Pending` to `permits` and declaring its record. Recompilation then identifies this switch as incomplete.

This protects source code you recompile together. Independently updated binaries can still encounter runtime matching failures. Guards also express handling rules, not constructor validation: this example does not prohibit negative amounts.

## Key takeaways

- Type patterns eliminate repeated type tests and casts.
- Record patterns expose components within a checked match.
- Guards refine cases; ordering still matters.
- Exhaustiveness supports safer changes, while null needs an explicit policy.

**Next: [Bound context and concurrent work](./03-scoped-values-and-structured-concurrency.md)**
