# 2. Memory, expiration, and persistence

> **TL;DR:** Explain separately why a key disappears, how memory is reclaimed, and which acknowledged writes survive a failure.

## Expiration follows time

A **TTL**, or time to live, limits a key's lifetime:

```text
SET session:42 active EX 60
```

`EX 60` sets a 60-second expiry in the same operation as the write. Redis stores expiry deadlines as absolute timestamps.

Redis removes expired keys through two mechanisms:

- **Passive expiration:** accessing an expired key discovers and removes it.
- **Active expiration:** periodic, bounded work finds expired keys even without client access.

A deadline makes the key logically expired; physical cleanup need not happen at that exact instant. Restarting does not reset its lifetime. [Expiration internals](https://redis.io/docs/latest/commands/expire/).

## Eviction follows memory pressure

**Eviction** removes keys to satisfy a memory policy. `maxmemory` configures Redis's memory limit for that policy.

| Policy family | Selection rule |
| --- | --- |
| `allkeys-lru` | Approximate least recently used keys across the keyspace. |
| `allkeys-lfu` | Approximate least frequently used keys across the keyspace. |
| `volatile-*` | Consider only keys with expiration deadlines. |
| `noeviction` | Reject memory-growing writes when the limit cannot be satisfied; reads can continue. |

**LRU** tracks recency; **LFU** tracks frequency. Redis approximates these policies using sampling and compact metadata. An unexpired key can therefore disappear through eviction. [Eviction documentation](https://redis.io/docs/latest/develop/reference/eviction/).

Treat `maxmemory` as a policy setting, not a strict ceiling on total process memory. Buffers, memory-management overhead, and persistence work require headroom.

## Persistence follows the failure model

**Persistence** stores recoverable data on disk. Compare these mechanisms:

| Mechanism | How it works | Main tradeoff |
| --- | --- | --- |
| RDB: Redis Database snapshot | Save a point-in-time dataset. | Compact recovery image; writes since that snapshot may be lost. |
| AOF: append-only file | Record writes for recovery. | More frequent recovery points; extra write and synchronization cost. |

With AOF, **fsync** requests that buffered data reach durable storage. `appendfsync everysec` normally leaves roughly a second of recent writes exposed to loss. That is not an unconditional bound under storage stalls or unusual settings.

`appendfsync always` synchronizes each appended batch before replies, increasing durability at a latency cost. `appendfsync no` leaves synchronization timing to the operating system. AOF must be enabled for these settings to matter. [Persistence documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/).

## Background work still has costs

`BGSAVE` creates a child process using **fork**, which gives the child a view of the parent's memory. **Copy-on-write** initially shares memory pages, then copies pages changed by the parent.

Think of photographing a notebook while someone keeps editing it. Changed pages need separate versions until the photograph finishes.

Fork can pause the parent. Heavy writes can increase copied memory significantly. [Redis's latency guide](https://redis.io/docs/latest/management/optimization/latency/) explains the fork and copy-on-write costs.

AOF rewriting produces a compact recovery representation while new writes continue. As of November 2026, modern Redis uses base and incremental AOF files managed together. [Persistence mechanics](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/).

## Worked failure question

You acknowledge a write, then the machine loses power before its next AOF synchronization. Can that write disappear after restart? Yes: execution and acknowledgment do not necessarily mean durable storage.

If deleting a huge collection causes latency, consider `UNLINK`. It detaches the key promptly and frees allocations asynchronously, reducing work on the command thread. [UNLINK reference](https://redis.io/docs/latest/commands/unlink/).

## Key takeaways

- TTL expiration and memory eviction solve different problems.
- State the persistence configuration before promising durability.
- Background snapshots can still cause pauses and memory growth.
- Recovery files and replicas are different protections.

**Next: [Replication, clustering, and interview drills](./03-replication-clustering-and-drills.md)**
