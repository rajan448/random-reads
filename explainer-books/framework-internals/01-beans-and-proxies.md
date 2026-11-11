# 1. Bean creation and proxies

> **TL;DR:** Spring creates your objects, connects their dependencies, and wraps selected objects to intercept method calls.

## Objects with a manager

A **bean** is an object managed by Spring. The **application context** is the container managing those beans.
**Dependency injection** means the container supplies collaborators, usually through constructor arguments.

Think of a restaurant manager assembling a kitchen. Each cook receives the equipment needed before service starts.
Your service similarly receives its database helper before handling requests.

Spring Boot adds **auto-configuration**: conditional setup based on available libraries, properties, and existing beans.
The **classpath** is Java's collection of available classes and libraries.
Adding a database library makes related setup possible; the matching conditions still need to pass.
Providing your own bean can make matching auto-configuration back off. [Boot auto-configuration](https://docs.spring.io/spring-boot/reference/using/auto-configuration.html).

## From blueprint to callable service

A **bean definition** is a blueprint describing how to create an object. A typical creation path looks like this:

1. Register definitions from configuration and component scanning, which discovers annotated application classes.
2. Apply definition processors, hooks that can change those blueprints.
3. Construct an object and supply its dependencies.
4. Run initialization callbacks, methods called while preparing the object.
5. Let object processors wrap eligible beans, then expose the resulting reference.

`BeanFactoryPostProcessor` changes definitions. `BeanPostProcessor` operates on objects around initialization.
An **auto-proxy creator** is an object processor that selects eligible beans for wrapping.
This is a simplified path; individual processors can participate earlier. [Container extension points](https://docs.spring.io/spring-framework/reference/core/beans/factory-extension.html).

## A wrapper around your method

A **proxy** is an object standing in for another object, called its target.
An **interceptor** runs additional behavior around a call. **AOP**, aspect-oriented programming, applies shared behavior across selected methods.

```text
caller → service proxy → transaction interceptor → service target
                              ↓
                       transaction manager
```

The proxy selects a chain of interceptors. That chain eventually invokes your method.
Transaction annotations are metadata, instructions interpreted by this infrastructure; annotations alone do not execute transaction code.

Two proxy mechanisms matter:

| Mechanism | How it stands in | Constraint |
| --- | --- | --- |
| JDK dynamic proxy | Implements Java interfaces | Calls use methods exposed through those interfaces. |
| CGLIB proxy | Generates a subclass of your class | Cannot override `final` or `private` methods, or subclass a `final` class. |

JDK means Java Development Kit; CGLIB names the subclass-generation library.
[Spring proxy mechanisms](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html).

As of November 2026, Boot's AOP auto-configuration defaults to class-based proxies.
`spring.aop.proxy-target-class=false` requests interface proxies where applicable. [Boot AOP](https://docs.spring.io/spring-boot/reference/features/aop.html).

## The self-invocation trap

**Self-invocation** means one target method calls another on the same object:

```java
public void checkout() {
    this.saveOrder();
}

@Transactional
public void saveOrder() {
    // Database writes.
}
```

If `checkout()` has no transaction, calling it through the proxy does not make `saveOrder()` transactional.
The internal call bypasses the wrapper. An existing outer transaction would still apply.
Move the boundary to an externally called service method, or call a separate injected bean.
Objects you construct with `new` also lack automatic Spring wrapping. [Transactional proxy behavior](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html).

## Key takeaways

- Separate bean definitions from actual objects.
- Follow the injected reference to understand interception.
- Check proxy boundaries when annotations seem ineffective.

**Next: [Transactions and connection pools](./02-transactions-and-connection-pools.md)**
