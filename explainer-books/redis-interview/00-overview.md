# Redis Internals for Interviews — Explainer Book

> **You'll come away knowing:** How Redis executes commands, represents values, manages memory, recovers data, and distributes ownership.
> **For:** A software engineer preparing for interviews · **Total reading time:** ~11 minutes

## Why this matters

Redis is a networked data store that keeps its working dataset in memory. You can use it as a cache, a fast copy of data available elsewhere, or as primary storage with appropriate safeguards.

Interviewers often move from “Why is it fast?” to “What fails?” Your goal is to connect each mechanism to its performance benefit and operational limitation.

## The 60-second version

- Core commands generally execute serially per server instance; auxiliary threads perform other work.
- Keys map to typed values. Their internal encodings balance memory cost and operation speed.
- Expiration removes old data; eviction handles memory pressure.
- Disk persistence and replication offer different protections. An acknowledgment alone does not establish crash durability.
- Sentinel coordinates failover. Cluster distributes keys across hash slots and supports failover.

## Key terms

| Term | Plain-English definition |
| --- | --- |
| RESP | Redis's request/reply encoding protocol. |
| Event loop / multiplexing | Repeatedly handle ready work / monitor many connections together. |
| Keyspace / hash table | Key-to-value mapping / a lookup structure organized into buckets by hashes. |
| Incremental rehashing | Move dictionary entries between tables in small steps. |
| Encoding | A value's actual memory representation. |
| SDS | Simple Dynamic String: a length-tracked, growable byte buffer. |
| Listpack / quicklist | Compact packed entries / linked groups of packed entries. |
| Skip list | Multiple linked levels supporting efficient ordered traversal. |
| Atomic execution | Other clients cannot interleave commands inside the operation. |
| Transaction / WATCH | A queued group executed together / a check that aborts after watched changes. |
| Pipelining | Send commands without waiting for each individual reply. |
| TTL / expiration | Time to live / removal after a deadline. |
| Eviction / maxmemory | Memory-pressure removal / its configured memory limit. |
| LRU / LFU | Least recently used / least frequently used selection. |
| Persistence / durability | Saving recoverable data / surviving a specified failure. |
| RDB / AOF | Redis Database snapshot / append-only recovery log. |
| fsync | Request that buffered writes reach durable storage. |
| Fork / copy-on-write | Create a child process / copy shared memory pages when changed. |
| Primary / replica | Write-serving owner / a follower of its updates. |
| Asynchronous replication | Copy updates without waiting for every replica before acknowledgment. |
| Backlog / resynchronization | Recent replication history / catching a replica up. |
| Failover / Sentinel | Replace an unavailable primary / Redis's non-clustered failover service. |
| Quorum | A configured number agreeing on a condition. |
| Sharding / hash slot | Distribute data ownership / a partition assigned to a primary. |
| CRC16 / hash tag | Cluster's slot hash / a selected key substring used for hashing. |
| MOVED / ASK | Ownership redirection / temporary migration redirection. |
| Hot key / big key | A heavily accessed key / a large value or collection. |

## Chapters

1. [01 — Command execution and data structures](./01-command-execution-and-data-structures.md) — Trace a command and justify operation costs.
2. [02 — Memory, expiration, and persistence](./02-memory-and-persistence.md) — Explain disappearing keys and crash recovery.
3. [03 — Replication, clustering, and interview drills](./03-replication-clustering-and-drills.md) — Explain scaling, failover, and practical bottlenecks.

## Scope and assumptions

You know client/server basics; C knowledge is unnecessary. Read the overview, then chapters 1–3, and rehearse the question table aloud.

This covers conventional Redis Open Source and core types, excluding modules, managed services, and full cache-design or distributed-lock treatments.

As of November 2026, details vary across releases. Verify encoding thresholds, defaults, and threading against your target version. Historical “ziplist” descriptions may differ.
