# 3. Memory, speed, and serving tradeoffs

> **TL;DR:** Serving performance depends on model weights, growing request state, hardware, and how the server schedules work.

## Separate three kinds of memory

**Serving** is the system that loads a model, accepts requests, schedules computation, and returns results. A **GPU**, or graphics processing unit, accelerates its large numerical operations.

For a dense model, where the usual layers use their weights each step:

```text
approximate weight memory = parameter count × bytes per parameter
total memory also includes KV caches, temporary values, and runtime overhead
```

An illustrative 8-billion-parameter model stored at 16 bits uses about 16 GB for weights alone. Here, GB means one billion bytes.

**Quantization** represents numbers with fewer bits. Four-bit weights have a raw size of about 4 GB for the same parameter count. Extra quantization metadata, temporary storage, and the KV cache increase actual requirements. Accuracy and speed depend on the method and hardware. [Quantization overview](https://huggingface.co/docs/transformers/en/quantization/overview).

## Estimate request-state memory

For a standard full-attention transformer with the same cache layout in each layer:

```text
KV bytes ≈ 2 × layers × cached tokens × KV heads
             × dimensions per head × bytes per cached number
```

The factor two accounts for keys and values. KV heads count stored key/value heads, which may differ from query heads.

For one sequence with 32 layers, 4,096 cached tokens, 8 KV heads, 128 dimensions per head, and two-byte numbers:

```text
2 × 32 × 4096 × 8 × 128 × 2 = 536,870,912 bytes
                           = 512 MiB
```

MiB means 1,048,576 bytes. Ten separate sequences of that length need roughly 5 GiB of cache, before overhead or sharing. GiB means 1,024 MiB. Architecture and cache optimizations can change this calculation. [Inference-memory discussion](https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization).

## Measure waiting and generation separately

| Metric | What you measure |
| --- | --- |
| TTFT: time to first token | Request arrival until the first output token; includes queueing and prompt processing. |
| Inter-token latency | Time between successive output tokens. |
| Throughput | Total tokens or requests processed per second, with the workload specified. |
| End-to-end latency | Request arrival until completion. |

For 100 output tokens, a simplified estimate is `TTFT + 99 × average inter-token latency`. Streaming and buffering can change what a client observes.

Prefill often stresses computation throughput. Small-batch decoding often stresses **memory bandwidth**, the rate of moving bytes. These are tendencies, not universal laws; context length, architecture, and batch size matter. [NVIDIA's attention-performance analysis](https://developer.nvidia.com/blog/co-designing-ai-model-attention-for-fast-interactive-long-context-inference?trk=article-ssr-frontend-pulse_little-text-block).

## Schedule requests efficiently

**Batching** runs multiple sequences together. Like cooking several meals with shared equipment, it can improve resource use while increasing waiting or memory demand.

**Continuous batching** admits and removes requests as generation progresses rather than waiting for an entire fixed batch to finish. [Continuous-batching documentation](https://huggingface.co/docs/transformers/en/continuous_batching).

**Paged attention** organizes KV storage into blocks, reducing allocation waste. [Paged-attention design](https://docs.vllm.ai/en/latest/design/paged_attention/).

**Prefix caching** reuses cached computation for matching input prefixes. These techniques manage or reuse state; they do not remove next-token dependencies. [Prefix-caching example](https://docs.vllm.ai/en/latest/examples/features/automatic_prefix_caching/).

## Explain the tradeoff aloud

Why can a long prompt delay the first token? More input requires more prompt processing, unless matching computation is reused.

Why can longer output cost more? It adds generation steps and typically grows the cache.

Does a low temperature guarantee correctness? No. It concentrates selection around the model's favored predictions.

## Key takeaways

- Weight size alone does not determine whether a workload fits.
- Separate first-token waiting from subsequent generation speed.
- Higher throughput can come with latency or memory costs.
- Benchmark representative prompts, output lengths, and concurrency.

**Where to go deeper:** Trace the previous chapter's loop, then estimate memory and latency for two different workloads.
