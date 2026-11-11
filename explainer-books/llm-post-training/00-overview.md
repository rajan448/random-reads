# LLM Post-Training — Explainer Book

> **You'll come away knowing:** What post-training changes, how its main methods work, and when you might use it.
> **For:** Curious beginners · **Total reading time:** ~10 minutes

## Why this matters

A **large language model (LLM)** generates language using patterns learned during training. **Pretraining** gives it a broad foundation through large amounts of data, often using next-token prediction.

**Post-training** is further training that shapes that foundation toward desired tasks, skills, and behavior. It can help a model follow instructions, handle conversations, or solve particular problems more reliably.

Think of a broadly educated person practicing for a specific job. They need examples, feedback, and checks on their performance. This analogy helps explain the training stages; it does not imply human-like understanding.

## The 60-second version

- Pretraining builds a broad foundation; post-training adapts that foundation for intended uses.
- Demonstrations teach what good responses look like. Comparisons teach which responses people prefer.
- Rewards can teach success on checkable tasks, such as solving problems or writing working code.
- Training changes learned numerical settings. Ordinary answering uses those settings without updating them.
- Better behavior does not guarantee correct answers. You still need evaluation and access to relevant information.

Published examples include instruction-following improvements through human feedback and reasoning improvements through reinforcement learning. [InstructGPT paper](https://arxiv.org/abs/2203.02155), [DeepSeek-R1 paper](https://arxiv.org/abs/2501.12948)

## Key terms

Use this table as a reference; each chapter also explains terms when you need them.

| Term | Plain-English definition |
| --- | --- |
| LLM | Large language model: a model that generates language from learned patterns. |
| Pretraining | Broad initial training, often through next-token prediction. |
| Post-training | Further training for desired behavior and tasks. |
| Fine-tuning | Adapting an existing model through additional training. |
| Weights / parameters | Learned numerical settings that shape model outputs. |
| Token | A piece of text, such as part of a word. |
| SFT | Supervised fine-tuning: learning from inputs and desired outputs. |
| Loss | A number measuring training error. |
| Backpropagation | Calculating how parameters contribute to training error. |
| Optimizer | The procedure that adjusts parameters using those calculations. |
| Preference learning | Learning from judgments comparing responses. |
| RL / reward | Reinforcement learning: improving behavior using scores called rewards. |
| RLHF | Reinforcement learning guided by human feedback. |
| Reward model | A separate learned model that scores responses. |
| DPO | Direct Preference Optimization: training directly on preferred and rejected responses. |
| Evaluation | Measuring performance against chosen requirements. |
| Held-out test set | Examples reserved for evaluation, excluded from training. |
| Inference | Running a trained model to produce an answer. |
| Prompt | Instructions and context supplied with an answer request. |
| RAG | Retrieval-augmented generation: supplying retrieved documents as answer context. |
| LoRA | Low-Rank Adaptation: training small added parameter matrices. |
| Overfitting | Learning training examples too narrowly to handle unfamiliar cases well. |
| Hallucination | Generated content that is unsupported or incorrect. |
| Reward hacking | Exploiting a scoring rule without achieving its intended goal. |

## Chapters

1. [01 — The main learning methods](./01-learning-methods.md) — Understand examples, preferences, and rewards.
2. [02 — Follow a training run](./02-follow-a-training-run.md) — Walk through a delivery-support assistant example.
3. [03 — Uses, choices, and limitations](./03-uses-and-limitations.md) — Choose between training, prompting, retrieval, and tools.

## Scope and assumptions

You need no machine-learning background. This book focuses on text-generating models and practical concepts, without training code or equations.

As of November 2026, the methods here are established examples, not a universal recipe for every model. Training methods evolve quickly; check updated research before implementing a pipeline.

## Key takeaways

- Post-training is a family of methods, not one compulsory step.
- Separate what a model knows, how it behaves, and what information it can access.
- Judge success on examples the model did not train on.
