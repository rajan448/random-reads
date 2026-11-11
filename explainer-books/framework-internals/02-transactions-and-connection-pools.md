# 2. Transactions and connection pools

> **TL;DR:** A transaction manager coordinates database work; HikariCP lends the connections used for that work.

## Trace a transfer

A **transaction** groups database changes into a unit. **Commit** accepts those changes; **rollback** undoes them.
**JDBC**, Java Database Connectivity, is Java's database access API.
A **DataSource** supplies connections; a **transaction manager** coordinates their transaction lifecycle.

Assume one database, Spring's JDBC transaction manager, and an external call through a Spring service proxy:

```java
@Transactional
public void transfer() {
    accounts.debit();
    accounts.credit();
}
```

This schematic example assumes both account operations use `JdbcTemplate`, Spring's JDBC helper, with the same DataSource.

1. The interceptor reads transaction settings and asks the manager to begin or join a transaction.
2. For a new transaction, the manager obtains a connection and disables **auto-commit**, which otherwise commits statements individually.
3. The manager binds the connection to the current thread, the executing stream of work.
4. `JdbcTemplate` reuses that bound connection for both operations.
5. The manager commits or rolls back, then cleans up and releases the connection when this transaction completes.

This binding uses thread-local storage: data associated with the executing thread.
Raw `DataSource.getConnection()` calls need not reuse it; prefer Spring's helpers.
Newly started threads do not inherit this transaction automatically. [Connection management](https://docs.spring.io/spring-framework/reference/data-access/jdbc/connections.html), [transaction annotations](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html).

Physical acquisition can happen later with lazy connection wrappers or some object-relational mapper (ORM) configurations.
An ORM maps objects to database tables.

## Boundaries and failure rules

**Propagation** describes how a method participates when a transaction already exists.

| Setting | Database behavior |
| --- | --- |
| `REQUIRED` | Join the existing transaction, or begin one. |
| `REQUIRES_NEW` | Suspend the outer scope; use an independent transaction and usually another connection. |
| `NESTED` | Use a savepoint, a rollback marker inside one transaction, when supported. |

With `REQUIRED`, an inner method can mark the shared transaction **rollback-only**, meaning it cannot commit.
Catching its exception outside that method does not clear the marker; commit can raise `UnexpectedRollbackException`.
[Propagation rules](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html).

As of November 2026, ordinary defaults roll back on escaping `RuntimeException` and `Error`, but not checked exceptions.
Checked exceptions are those Java normally requires you to catch or declare.
Use `rollbackFor` or configured global rules when your application needs different behavior.
Swallowing an exception inside your method can hide it from the interceptor. [Rollback rules](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/rolling-back.html).

`readOnly=true` is an optimization hint, not a universal write prohibition.
A local database transaction cannot undo a remote payment or email. [Transaction settings](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html).

## Borrowing from HikariCP

A **connection pool** keeps database connections available for reuse, like a library lending books.
As of November 2026, Boot prefers HikariCP when available; its JDBC and data-JPA starters include it.
A starter is a dependency bundle. [Boot database setup](https://docs.spring.io/spring-boot/reference/data/sql.html).

HikariCP returns a connection wrapper. Its `close()` normally returns the connection to the pool, preserving the physical database session.
Spring releases transaction-bound connections after transaction completion. [HikariCP connection source](https://github.com/brettwooldridge/HikariCP/blob/dev/src/main/java/com/zaxxer/hikari/pool/ProxyConnection.java).

| Boot property under `spring.datasource.hikari` | Purpose |
| --- | --- |
| `maximum-pool-size` | Maximum total connections, including borrowed and idle ones. |
| `minimum-idle` | Target number of available idle connections. |
| `connection-timeout` | Maximum wait to borrow; does not limit query execution. |
| `max-lifetime` | Retirement age; in-use connections retire after return. |

As of November 2026, upstream defaults include a pool maximum of 10 and a 30,000-millisecond borrowing timeout.
Check your resolved version and overrides. [HikariCP configuration](https://github.com/brettwooldridge/HikariCP#configuration-knobs-baby).

## A capacity example

Hypothetically, 100 transfers/second holding connections for 0.05 seconds average about five borrowed connections.
Adding a one-second remote call inside each transaction raises demand to about 105. This estimates average occupancy, not safe sizing.

Twenty outer transactions can also occupy a twenty-connection pool while their `REQUIRES_NEW` calls wait for extra connections.
Waiting eventually times out. Shorten boundaries and measure database capacity before enlarging pools.

## Key takeaways

- Put boundaries around coherent database work.
- Track connection holding time as well as query time.
- Budget connections across every application instance.

**Next: [Startup costs and debugging](./03-startup-costs-and-debugging.md)**
