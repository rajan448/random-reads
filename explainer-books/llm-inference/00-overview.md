# LLM Inference — Explainer Book

> **You'll come away knowing:** How prompts become tokens, why generation has two phases, and what limits speed.
> **For:** Curious beginners and engineers · **Total reading time:** ~11 minutes

## Why this matters

**Inference** uses trained models to compute outputs. Ordinary language-model generation repeatedly predicts a next token from available context.

Understanding the loop explains streaming, prompt delays, memory use, and serving tradeoffs.

## The 60-second version

- Text and chat markers become token IDs, then numerical representations.
- Learned transformer layers turn those representations into next-token scores.
- Prefill processes known input and produces the first token's scores.
- A decoding loop selects tokens and extends the sequence, reusing cached attention keys and values.
- Weights normally stay fixed. Temporary state, memory traffic, and request scheduling determine much of the runtime cost.

Sources include official [generation](https://huggingface.co/docs/transformers/en/llm_tutorial) and [caching](https://huggingface.co/docs/transformers/en/cache_explanation) documentation.

## The request's journey

```text
messages → chat formatting → token IDs → prefill → next-token scores
                                                    ↓
                                                select token
                                                    ↓
                                        stream output / check stop
                                                    ↓
                              process selected token using KV cache
                                                    ↓
                                         new next-token scores ↺
```

## Key terms

| Term | Plain-English definition |
| --- | --- |
| LLM | Large language model: a network modeling language patterns. |
| Training / inference | Learn model parameters / use them to compute outputs. |
| Weights / parameters | Learned numbers controlling the model's computations. |
| Forward pass / gradient | Compute outputs / a parameter-adjustment signal during training. |
| Token / vocabulary | A text unit / the tokenizer's set of recognized tokens. |
| Tokenizer / chat template | Text-to-ID converter / formatting for message roles and boundaries. |
| Embedding / vector | A token's numerical representation / a list of numbers. |
| Transformer | A network using attention and other layers. |
| Causal attention | Combine information without accessing future positions. |
| Query, key, value | Learned representations calculating relevance and carrying information. |
| Attention head | One learned attention projection and mixing operation. |
| Feed-forward network | A network transforming each position's vector. |
| Normalization / residual connection | Rescale values / add earlier representations to updates. |
| Logits / softmax | Unnormalized token scores / a function converting them into probabilities. |
| Prefill | Process known prompt tokens and initialize generation state. |
| Autoregressive decoding | Generate using input and previously generated tokens. |
| KV cache | Reusable attention keys and values for processed positions. |
| Greedy decoding / sampling | Choose the highest score / draw from a distribution. |
| Temperature / top-p | Adjust distribution sharpness / sample from a cumulative-probability shortlist. |
| EOS / streaming | End-of-sequence token / incremental delivery of output. |
| Context | Input and generated tokens available to the model. |
| Serving / GPU | Request-processing infrastructure / a processor suited to parallel numerical work. |
| Quantization | Represent model numbers with fewer bits. |
| TTFT / inter-token latency | Wait for first token / time between later tokens. |
| Throughput / memory bandwidth | Work completed per second / bytes transferred per second. |
| Batching / continuous batching | Process requests together / update the group as requests finish or arrive. |
| Paged attention / prefix caching | Block-based KV storage / reuse of matching-prefix computation. |

## Chapters

1. [01 — From text to next-token scores](./01-tokens-and-transformers.md) — Follow tokenization, attention, and output scoring.
2. [02 — Prefill, decoding, and the KV cache](./02-prefill-decoding-and-kv-cache.md) — Trace one request and its generation loop.
3. [03 — Memory, speed, and serving tradeoffs](./03-memory-speed-and-serving.md) — Work through memory estimates and performance questions.

## Scope and assumptions

Read this overview, then chapters 1–3. Basic arithmetic is enough.

This covers conventional decoder-only transformer text generation. Other architectures, speculative generation, multimodal inputs, tools, training mathematics, and deployment setup are outside its scope.

As of November 2026, inference software evolves quickly. Verify runtime features against current documentation.
