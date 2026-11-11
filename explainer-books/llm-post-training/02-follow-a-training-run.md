# 2. Follow a training run

> **TL;DR:** Define success, prepare examples, update the model, and test whether the changes help on unfamiliar inputs.

## A support assistant example

Imagine you want an assistant that handles delivery problems. Your desired behavior is concrete: acknowledge the problem, ask for missing information, and avoid unsupported promises.

The following examples and numbers are invented for teaching. They are not measured results or suggested dataset sizes.

## 1. Define success before training

You decide what a good response must do. For a missing package, it should request an order number. It should not promise a refund without checking eligibility.

Prepare evaluation questions, including unusual wording and incomplete information. **Evaluation** means measuring the model against your chosen requirements.

## 2. Build and separate the data

Create examples like this:

```text
Customer: My package never arrived.
Desired response: I'm sorry your package hasn't arrived.
Please share your order number so we can check its status.
```

Include varied situations, such as damaged items and uncertain delivery dates. Check examples for accuracy and consistency.

Keep a **held-out test set**: questions reserved for evaluation and never used for training. Keep closely related examples together when splitting data. Otherwise, near-duplicates can make apparent progress misleading.

## 3. Update the model

Use **supervised fine-tuning (SFT)**: training on inputs paired with desired responses.

For each example, the model assigns probabilities to the response's tokens, or text pieces. The training program measures error and adjusts numerical settings called weights.

Suppose the target next token receives probability 0.20 before an update and 0.24 afterward. That illustrates the direction of learning. Actual updates affect many predictions and do not guarantee improvement on every example.

You repeat the process across groups of examples. The goal is useful behavior on new questions, not just reproduction of training answers.

## 4. Optionally add comparisons

For the same customer message, compare two answers:

| Candidate | Response | Judgment |
| --- | --- | --- |
| A | “You will definitely receive a refund.” | Reject: unsupported promise |
| B | “Please share your order number so we can investigate.” | Prefer: useful next step |

You can train using such pairs through **Direct Preference Optimization (DPO)**, a method for learning relative response preferences. This is an illustrative application of the method. [DPO paper](https://arxiv.org/abs/2305.18290)

## 5. Compare and deploy

Test the original and updated models on the same held-out questions. Check helpfulness, unsupported promises, and performance on other tasks you still need.

If the updated model passes your requirements, save its learned settings and use it to answer requests. This answering stage is **inference**. Ordinary inference does not update the weights.

User feedback only changes future weights if you collect it and run another training process.

## Key takeaways

- Define the behavior you want before collecting examples.
- Separate training examples from evaluation questions.
- Measure improvements and regressions together.
- Deployment uses the learned settings; further learning requires another training process.

**Next: [Uses, choices, and limitations](./03-uses-and-limitations.md)**
