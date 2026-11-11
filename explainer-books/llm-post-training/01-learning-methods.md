# 1. The main learning methods

> **TL;DR:** You can teach through example answers, comparisons between answers, or rewards for successful attempts.

## Start with a pretrained model

A large language model, or LLM, generates language using learned numerical settings called **weights** or **parameters**. During **pretraining**, it learns from large collections of text, often by predicting the next text piece.

These pieces are **tokens**: words, parts of words, or punctuation. Predicting them teaches language patterns, facts, and useful skills. However, continuing text and responding helpfully to instructions are different training goals.

**Post-training** means further training that shapes an existing model for desired tasks and behavior. **Fine-tuning** means adapting an existing model through additional training; it overlaps substantially with post-training.

Think of pretraining as a broad education and post-training as supervised practice for a job. The analogy describes the stages, not human-like understanding.

## Supervised fine-tuning: show good answers

**Supervised fine-tuning (SFT)** trains on inputs paired with desired outputs.

For example, you supply a customer question and a carefully written support response. During training, the model learns to make the target response's tokens more likely.

A **loss** is a numerical measure of training error. **Backpropagation** calculates how the weights contribute to that error. An **optimizer** uses those calculations to adjust the weights.

Many small adjustments across examples can teach reusable patterns. InstructGPT used demonstrations before preference-based training. [InstructGPT paper](https://arxiv.org/abs/2203.02155)

## Preference learning: compare answers

**Preference learning** uses judgments such as “answer A is better than answer B.”

**Reinforcement learning (RL)** improves behavior using scores called **rewards**. **Reinforcement learning from human feedback (RLHF)** uses human judgments to guide that process.

In a classic RLHF setup, comparisons train a **reward model**, a separate model that scores responses. RL then updates the answering model toward higher scores, while limiting excessive changes. [InstructGPT methods](https://arxiv.org/html/2203.02155v1#S3.SS1)

**Direct Preference Optimization (DPO)** instead trains directly on preferred and rejected response pairs. It changes their relative likelihoods compared with a reference model, without training a separate reward model. DPO and classic RLHF are alternative approaches; you do not always need both. [DPO paper](https://arxiv.org/abs/2305.18290)

## Learning from checkable results

Sometimes you can check success automatically. Code can pass or fail tests; a math answer can match a verified result.

RL can use these checks as rewards. You need fewer subjective judgments for outcomes you can verify. This can improve problem-solving, although passing limited checks does not prove general correctness. [DeepSeek-R1 paper](https://arxiv.org/abs/2501.12948)

## Key takeaways

- SFT supplies examples of the desired answer.
- Preference learning supplies comparisons between answers.
- RL supplies rewards for generated attempts.
- Training changes learned parameters, rather than merely adding instructions to a conversation.
- There is no mandatory sequence containing every method.

**Next: [Follow a training run](./02-follow-a-training-run.md)**
