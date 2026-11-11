# 2. Prefill, decoding, and the KV cache

> **TL;DR:** Process the prompt once, choose a token, then repeatedly process the chosen token while reusing cached attention information.

## Prefill processes known input

**Prefill** runs the prompt through the model and prepares information needed for generation.

Because you already know every prompt token, the model can process many prompt positions in parallel within each layer. The causal attention mask still prevents each position from using later positions.

Layers retain their dependencies: a later layer needs the earlier layer's results. “Parallel prompt processing” does not mean every calculation happens at once.

The last prompt position's output scores the first generated token. You can select that token directly from prefill's output. [NVIDIA's inference explanation](https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization) distinguishes prefill and subsequent decoding.

## Decode extends the sequence

**Autoregressive decoding** repeatedly conditions generation on earlier input and generated tokens. In the ordinary one-token loop:

| Stage | New input processed | What its output predicts |
| --- | --- | --- |
| Prefill | All prompt tokens | First generated token, A |
| Decode step 1 | Token A | Next token, B |
| Decode step 2 | Token B | Next token, C |

You cannot know B's input until A is selected. The hardware can parallelize calculations inside a step and across requests, while this sequence dependency remains.

Generation ends at an **EOS**, or end-of-sequence token, a configured stopping condition, or a length limit. Converting the resulting token IDs into readable text is a separate tokenizer operation. [Generation guide](https://huggingface.co/docs/transformers/en/llm_tutorial).

## The KV cache saves repeated work

A **key-value cache**, or KV cache, stores attention keys and values already computed for earlier positions, at each layer.

During decoding, the new position computes its own query, key, and value. Its query attends to cached keys and values plus the new position's information. The new key and value extend the cache.

Imagine keeping labeled notes from earlier pages rather than rereading the book to recreate them. You still consult those notes; caching does not eliminate attention over history.

The cache contains intermediate numbers, not finished answers. It differs from model weights and from a cache of complete responses. [KV-cache explanation](https://huggingface.co/docs/transformers/en/cache_explanation).

## Choose the next token

**Greedy decoding** selects the highest-scoring token. **Sampling** draws a token from a probability distribution.

**Temperature** adjusts logits before softmax. A lower positive temperature makes the distribution more concentrated; a higher temperature flattens it. **Top-p sampling** retains the smallest high-probability group reaching a chosen cumulative probability, then samples within it.

Suppose these are illustrative next-token probabilities:

| Candidate | Probability |
| --- | ---: |
| ` Paris` | 0.70 |
| ` Lyon` | 0.10 |
| All other tokens combined | 0.20 |

Greedy selection chooses ` Paris`. Unrestricted sampling can choose something else. These numbers illustrate mechanics, not measured model output. [Decoding strategies](https://huggingface.co/docs/transformers/en/generation_strategies).

## Put the loop together

```text
cache, logits = prefill(prompt_tokens)
token = select(logits)

while token is not EOS and output_limit_not_reached:
    emit(token)
    logits, cache = model_step(token, cache)
    token = select(logits)
```

This is explanatory pseudocode, not a library API. It omits batching, other stopping rules, and output buffering. **Streaming** sends generated output incrementally; it improves when you see progress without removing the underlying calculations.

## Key takeaways

- Prefill can produce the first token's scores.
- Ordinary decoding extends one dependent sequence step by step.
- KV caching reuses earlier attention work and consumes memory.
- Sampling settings change selection, not model weights.

**Next: [Memory, speed, and serving tradeoffs](./03-memory-speed-and-serving.md)**
