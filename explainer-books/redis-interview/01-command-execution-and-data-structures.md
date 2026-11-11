# 1. Command execution and data structures

> **TL;DR:** Redis combines memory access, efficient representations, and serialized command execution; expensive commands still delay everyone sharing that execution thread.

## Follow one command

Imagine Redis as a fast librarian serving many waiting readers. The librarian handles short requests quickly, but one enormous request creates a queue.

For `GET user:42`, the logical path is:

```text
client → network read → parse command → locate key
       → check expiry and type → read value → queue reply
```

The client uses **RESP**, the Redis Serialization Protocol, to encode requests and replies.

An **event loop** repeatedly handles ready connections and scheduled work. **I/O multiplexing** lets the operating system report which connections are ready, without assigning each client a dedicated thread. Linux commonly uses the `epoll` readiness mechanism. [Redis's event-library documentation](https://redis.io/docs/latest/operate/oss_and_stack/reference/internals/internals-rediseventlib/) explains this architecture.

Core commands generally execute on one main thread per server instance. Other threads can handle network I/O or background work. Therefore, “Redis has only one thread” is inaccurate. A slow command can still delay other clients' command execution. [Redis's latency guide](https://redis.io/docs/latest/management/optimization/latency/) explains this consequence.

## Find the key, then interpret its value

Conceptually, the **keyspace** maps key names to typed values using hash-table dictionaries. A **hash table** maps a computed hash to a storage bucket. Collisions occur when different keys share a bucket.

Expected lookup cost is `O(1)`: roughly constant as key count grows, excluding key hashing and returned bytes. `O(N)` means work grows linearly with `N`; `O(log N)` grows much more slowly.

**Incremental rehashing** moves entries between old and new tables in steps during resizing. This spreads migration work rather than moving every entry at once. It does not eliminate allocation costs or all latency spikes. See [Redis's dictionary implementation](https://github.com/redis/redis/blob/8.0/src/dict.c).

## Separate logical types from physical encodings

An **encoding** is a type's actual memory representation. As of November 2026, these are useful interview patterns; exact choices depend on release, size, and configuration.

| Logical type | Typical representations and purpose |
| --- | --- |
| String: a byte sequence | Integer encoding, or SDS: a length-tracked, growable byte buffer. Small strings may share one allocation with object metadata. |
| Hash: field/value pairs | Compact listpack or hash table. |
| List: ordered elements | Listpack or quicklist: linked nodes containing packed groups. |
| Set: unique members | Integer set, listpack, or hash table. |
| Sorted set: unique members with scores | Compact listpack, or a skip list plus dictionary. |

A **listpack** stores entries compactly together, reducing pointer overhead. A **skip list** uses multiple linked levels to accelerate ordered traversal. The sorted-set dictionary supports member lookup; the skip list supports ordering and ranks. [Encoding reference](https://redis.io/docs/latest/commands/object-encoding/), [sorted-set documentation](https://redis.io/docs/latest/develop/data-types/sorted-sets/).

For a leaderboard, `ZADD board 120 alice` maintains Alice's score and order. A single-member insertion is `O(log N)`; returning `M` ranked members typically costs `O(log N + M)`.

## Atomicity and batching

**Atomic execution** means other clients cannot interleave commands inside one operation. `INCR counter` avoids the race between a separate read and write.

`MULTI` queues commands; `EXEC` executes the queued transaction without interleaving. Runtime errors do not roll back successful commands. `WATCH` aborts execution if watched keys change, allowing a retry. [Transaction semantics](https://redis.io/docs/latest/develop/using-commands/transactions/).

**Pipelining** sends several commands without waiting for each reply. It reduces network waiting and system-call overhead; it does not make the batch atomic. [Pipelining guide](https://redis.io/docs/latest/develop/using-commands/pipelining/).

## Key takeaways

- Explain the command path before listing performance claims.
- Distinguish logical types from memory encodings.
- State command complexity and result size.
- Separate atomic execution, transactions, and pipelining.

**Next: [Memory, expiration, and persistence](./02-memory-and-persistence.md)**
