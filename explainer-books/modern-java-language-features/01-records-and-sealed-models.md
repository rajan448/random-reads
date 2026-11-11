# 1. Model data with records and sealed types

> **TL;DR:** Use records for transparent data and sealed types for a controlled set of alternatives.

## Records make the data contract visible

A **record** declares its data in a header. Each named value is a **component**. Java supplies a constructor, component accessors, and `equals`, `hashCode`, and `toString` methods.

You call `order.id()`, rather than a generated `getId()`. Record classes are implicitly `final`: another class cannot extend them. They can implement interfaces.

Two records of the same type compare equal when their corresponding components compare equal. This reduces mistakes from handwritten equality methods. See the [record class guide](https://docs.oracle.com/en/java/javase/21/language/records.html).

Think of a record as a labeled shipping form. Its shape is part of its public contract. Choose an ordinary class when you need hidden, changing internal state.

## Validate once at construction

An **invariant** is a condition every valid instance must satisfy. A **compact constructor** checks or normalizes record arguments before Java assigns them to fields.

Save this complete Java 21 example as `ModelDemo.java`:

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

public class ModelDemo {
    record Order(String id, List<String> items) {
        Order {
            Objects.requireNonNull(id, "id");
            if (id.isBlank()) {
                throw new IllegalArgumentException("blank id");
            }
            items = List.copyOf(items);
            if (items.isEmpty()) {
                throw new IllegalArgumentException("empty order");
            }
        }
    }

    sealed interface Outcome permits Accepted, Rejected {}
    record Accepted(Order order) implements Outcome {}
    record Rejected(String reason) implements Outcome {}

    public static void main(String[] args) {
        var original = new ArrayList<>(List.of("book"));
        var order = new Order("A1", original);
        original.add("pen");
        System.out.println(order.items()); // [book]
        System.out.println(order.equals(
                new Order("A1", List.of("book")))); // true
    }
}
```

Compile with `javac --release 21 ModelDemo.java`; run with `java ModelDemo`.

## Final fields do not freeze their objects

A record's component fields cannot be reassigned. A referenced collection can still change unless you protect it. This is **shallow immutability**.

`List.copyOf` produces an unmodifiable list, preserving this order when the caller changes its original list. It rejects a null list or null elements. Its elements are not recursively copied; mutable elements would need their own protection. [List.copyOf contract](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html#copyOf(java.util.Collection))

Arrays need special care: generated record equality uses an array's reference equality, rather than comparing its contents. Also remember that generated `toString` exposes components; avoid logging records containing secrets. [Record API contract](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html)

## Sealed types describe the allowed alternatives

A **sealed type** restricts its direct subtypes. The `permits` clause names them. Our `Outcome` has two alternatives, each carrying different data.

A permitted class must be `final`, `sealed`, or `non-sealed`. A `non-sealed` class reopens its branch to arbitrary subclasses. Records already satisfy the `final` requirement.

Permitted subtypes must share the sealed type's named module, or its package when using the unnamed module. Keeping the declarations together makes this example straightforward. [Sealed class rules](https://docs.oracle.com/en/java/javase/21/language/sealed-classes-and-interfaces.html)

This models alternatives more clearly than `boolean accepted` plus several nullable fields. It still permits null component values unless you validate them. A sealed hierarchy controls inheritance; it does not enforce every business rule.

## Key takeaways

- Records expose a deliberate data shape and consistent generated methods.
- Compact constructors centralize validation and defensive copying.
- Final references provide shallow immutability, not automatic thread safety.
- Sealed types make the intended alternatives visible to the compiler.

**Next: [Match and handle every alternative](./02-pattern-matching.md)**
