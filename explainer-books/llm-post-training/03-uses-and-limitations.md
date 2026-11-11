# 3. Uses, choices, and limitations

> **TL;DR:** Post-training helps shape repeatable behavior and skills; choose it when your measured problem calls for training.

## What you can use it for

**Post-training** is additional training that adapts an existing language model. You might use it to improve these behaviors:

| Use | Desired change | Example evaluation |
| --- | --- | --- |
| Instruction following | Respond in the requested way | Does the answer follow the requested format? |
| Customer support | Follow a consistent support process | Does it request missing details? |
| Document processing | Extract fields consistently | Are the extracted names and dates correct? |
| Safety behavior | Handle harmful or uncertain requests appropriately | Does it handle both ordinary and adversarial examples? |
| Coding and math | Solve more checkable problems | Does the code pass unseen tests? |
| Tool use | Produce suitable requests to external software | Is the chosen operation valid for the task? |

These are possible goals, not guaranteed outcomes. Demonstrations and human feedback have improved instruction following in published research. Rewards from checkable tasks have also supported reasoning improvements. [InstructGPT](https://arxiv.org/abs/2203.02155), [DeepSeek-R1](https://arxiv.org/abs/2501.12948)

## Choose the right intervention

A **prompt** is the instruction and context you provide when requesting an answer. Start by checking whether clearer instructions solve your problem.

**Retrieval-augmented generation (RAG)** retrieves relevant documents and supplies them as context for an answer. It gives a model access to external information. [Original RAG paper](https://arxiv.org/abs/2005.11401)

For our delivery assistant, consider three different needs:

- “Ask politely for an order number”: try a prompt, then training if behavior remains inconsistent.
- “What is this customer's delivery status?”: connect to the order system.
- “What is our latest refund policy?”: retrieve the current policy document.

My practical recommendation is to match the intervention to the failure you measure. Post-training and retrieval can work together: one shapes behavior, while the other supplies relevant information. A deployed RAG system need not retrain the model for each document update.

## Training without changing every original weight

**LoRA**, short for Low-Rank Adaptation, trains small additional parameter matrices while keeping the original model weights fixed. These additional learned settings change the model's effective behavior. This can reduce training memory and the number of parameters you update. [LoRA paper](https://arxiv.org/abs/2106.09685)

LoRA describes how you make updates. It does not replace the need for good examples, a training objective, or evaluation.

## What can go wrong

**Overfitting** means learning training examples too narrowly and performing poorly on unfamiliar inputs. **Hallucination** means generating unsupported or incorrect content.

**Reward hacking** means exploiting weaknesses in a score instead of achieving the intended goal. For example, rewarding long responses can encourage unnecessary explanations. These terms describe different failures; check for each separately.

> ⚠️ Post-training does not guarantee truth, eliminate harmful behavior, or keep factual knowledge current.

## Key takeaways

- Use post-training for measured behavior and skill improvements.
- Use retrieval or tools when answers require external information.
- Evaluate unfamiliar cases and previously useful capabilities.
- Treat better training scores as evidence to investigate, not proof of deployment quality.

**Go deeper:** Revisit the worked example and design ten evaluation questions before thinking about training code.
