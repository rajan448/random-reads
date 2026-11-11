# 1. From text to next-token scores

> **TL;DR:** A language model converts text into numbers, transforms them through learned layers, and scores possible next tokens.

## Inference uses learned weights

A **large language model**, or LLM, is a neural network trained to model language patterns. **Weights**, also called parameters, are the learned numbers governing its computations.

**Training** adjusts weights using examples. **Inference** uses those weights to compute outputs. In ordinary inference, reading your prompt changes temporary computations, not model weights.

A **forward pass** runs inputs through the network. Training also computes **gradients**, signals describing how to adjust weights, through a backward computation. Standard inference skips that gradient computation. [PyTorch's inference-mode documentation](https://docs.pytorch.org/docs/stable/generated/torch.autograd.grad_mode.inference_mode.html) describes disabling this machinery.

Think of weights as a learned recipe. Your prompt supplies ingredients; preparing a new dish does not rewrite the recipe.

## Text becomes token IDs

A **token** is a text unit, which might be a word, word fragment, punctuation, or byte-related unit. A **tokenizer** converts text into token IDs from a fixed **vocabulary**, the set of recognized tokens.

For illustration, invent this tiny tokenizer:

```text
"The capital of France is"
→ ["The", " capital", " of", " France", " is"]
→ [11, 24, 36, 48, 59]
```

These chunks and IDs are made up. Actual segmentation depends on the model's tokenizer. Spaces can be part of tokens. [Tokenization overview](https://huggingface.co/docs/transformers/en/tokenizer_summary).

A chat service first applies a **chat template**, formatting messages with role and boundary markers. Conversation history can therefore become part of the model's input token sequence. [Chat-template documentation](https://huggingface.co/docs/transformers/en/chat_templating).

## Tokens become contextual representations

An **embedding** maps a token ID to a vector, a list of numbers. Position information distinguishes different arrangements of the same tokens.

A **transformer** repeatedly updates these representations through layers. In the decoder-only models covered here, **causal attention** lets a position use itself and preceding positions, while blocking later positions.

**Attention** mixes information from accessible positions using computed relevance weights. Each position produces:

- A **query**: a learned representation used to request relevant information.
- A **key**: a learned representation used to calculate relevance.
- A **value**: information combined according to that relevance.

An **attention head** performs one such mixing operation. Multiple heads allow different learned projections. These are mathematical computations, not searches through a document database.

Other components include a **feed-forward network**, which transforms each position's vector; **normalization**, which rescales internal values; and **residual connections**, which add earlier representations to later updates. [The original transformer paper](https://arxiv.org/html/1706.03762v7) introduces these components. Modern model details vary.

## Representations become predictions

The final representation at the last input position is projected into **logits**, one unnormalized score per vocabulary token. **Softmax** converts scores into probabilities that sum to one.

For the invented prompt above, the model might assign high probability to the token `" Paris"`. A selection rule chooses the next token; that choice becomes input for further generation. [Text-generation guide](https://huggingface.co/docs/transformers/en/llm_tutorial).

High probability means the model favors a continuation. It does not certify that a factual claim is true.

## Key takeaways

- Standard inference uses weights without updating them.
- Tokens are model-specific units, not necessarily words.
- Attention builds representations from accessible context.
- The network produces scores; a selection rule produces a token.

**Next: [Prefill, decoding, and the KV cache](./02-prefill-decoding-and-kv-cache.md)**
