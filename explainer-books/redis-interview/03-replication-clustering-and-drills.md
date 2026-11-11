# 3. Replication, clustering, and interview drills

> **TL;DR:** Replication copies data, failover replaces a primary, and sharding spreads ownership; none automatically guarantees lossless writes.

## Follow a write across replicas

A **primary** accepts writes. A **replica** follows its update stream. Default replication is **asynchronous**: the primary can acknowledge before replicas apply the write.

A reconnecting replica requests **partial resynchronization** using its replication history and position. A **replication backlog**, a bounded recent-history buffer, can supply missing updates. If history is unavailable or incompatible, **full resynchronization** transfers a snapshot followed by updates.

Worked failure:

```text
primary applies W → client receives OK → primary fails
                 replica has not received W → replica promoted
```

The promoted replica lacks `W`. Replication therefore permits an acknowledged write to disappear during failover. Reading from replicas can also return stale data.

`WAIT` asks for replica acknowledgments of earlier writes on that connection. Inspect its returned count. It improves safety but does not guarantee lossless failover or disk durability. [Replication documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/).

## Sentinel handles failover

**Failover** promotes a replica when the primary is considered unavailable. **Sentinel** monitors a non-clustered Redis deployment, coordinates failover, and helps clients discover the current primary.

A **quorum** is the configured number of Sentinels agreeing that a primary is down. Authorizing failover additionally requires a majority of Sentinels. Sentinel does not divide the dataset across primaries. [Sentinel documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/).

## Cluster divides the keyspace

**Sharding** distributes data ownership. Redis Cluster maps keys to **16,384 hash slots**, partitions of the keyspace assigned to primaries:

```text
slot = CRC16(key) modulo 16384
```

`CRC16` is the hash calculation used here. A cluster-aware client routes requests using slot ownership. `MOVED` tells it the responsible node; `ASK` temporarily redirects a request during slot migration.

**Hash tags** select the key substring used for slot hashing. `cart:{42}` and `stock:{42}` share a slot because both hash `42`. Most multi-key operations require their keys to share a slot.

Cluster uses primary/replica failover and asynchronous replication. Network partitions can cause unavailability and lost writes; it does not provide unlimited availability with strong consistency. [Cluster specification](https://github.com/redis/docs/blob/main/content/operate/oss_and_stack/reference/cluster-spec.md).

Think of slots as delivery zones and primaries as depots. Reassigning zones changes ownership. Adding depots does not divide one overloaded address.

## Diagnose the bottleneck

A **hot key** receives concentrated traffic. A **big key** holds a large value or collection. Sharding alone does not split either key.

For slow key enumeration, prefer incremental `SCAN` over a large blocking `KEYS` request. A full scan remains `O(N)`, may return duplicates, and is not a consistent snapshot during changes. `COUNT` is a work hint. [SCAN reference](https://redis.io/docs/latest/commands/scan/).

Check `SLOWLOG GET` for slow command execution. It excludes network communication, so a clean slow log does not prove fast client responses. [SLOWLOG reference](https://redis.io/docs/latest/commands/slowlog/).

## Rehearse aloud

| Interview question | Your answer should include |
| --- | --- |
| Why is Redis fast? | Memory access, efficient structures, short commands, multiplexing; network and slow-command limits. |
| Does atomic mean durable? | No; interleaving and crash survival are separate properties. |
| Sentinel or Cluster? | Failover for one primary's dataset versus sharding plus failover. |
| Can an unexpired key disappear? | Eviction, explicit deletion, or recovery/failover that loses its write. |
| Does adding replicas scale writes? | Writes still execute on the primary; shards distribute write ownership. |

## Key takeaways

- Walk through failures using an explicit timeline.
- Separate replication, failover, sharding, and durability.
- Explain slot constraints before suggesting multi-key operations.
- Diagnose command time separately from network and queueing time.

**Where to go deeper:** Practice a rate limiter, leaderboard, and cache design. For each, name the structure, atomic operation, expiry policy, and failure behavior.
