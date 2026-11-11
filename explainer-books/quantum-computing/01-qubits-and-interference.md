# 1. Qubits, waves, and shared states

> **TL;DR:** You compute with quantum states by controlling amplitudes, whose phases determine how they combine.

## From bits to qubits

A **bit** stores an ordinary computer's binary value: 0 or 1. A **qubit** uses two distinguishable quantum states, labeled `|0⟩` and `|1⟩`.

The brackets simply mean “the quantum state labeled inside.” They do not describe a container.

A qubit can have a **superposition**, a combination of these states:

```text
state = a|0⟩ + b|1⟩
```

Here, `a` and `b` are **amplitudes**: mathematical weights used to calculate measurement probabilities. **Measurement** is the physical process that produces an observable result.

For the 0/1 measurement used throughout this book:

```text
probability of 0 = |a|²
probability of 1 = |b|²
```

The bars mean magnitude, or size. The two probabilities must add to one.

If both amplitudes equal `1/√2`, each outcome has probability one-half. Here, `√2` means the square root of two.

## Why a superposition differs from a coin toss

A hidden coin has an unknown result. A superposition carries information about how alternatives can interfere.

An amplitude has both magnitude and **phase**, an angle describing its orientation. Our examples use positive and negative amplitudes: two opposite phases.

Imagine two water waves meeting. Aligned crests reinforce each other; a crest meeting an equal trough cancels. **Interference** is this reinforcement or cancellation, applied here to amplitudes.

You combine amplitudes before calculating probabilities. Probabilities themselves never become negative. These rules explain why matching measurement probabilities need not imply matching quantum states. [IBM's quantum mechanics lesson](https://quantum.cloud.ibm.com/learning/en/courses/use-a-qc-today/quantum-mechanics-basics) develops this connection.

## When qubits share a state

Two qubits have four possible 0/1 measurement results: `00`, `01`, `10`, and `11`.

Consider this joint state:

```text
(|00⟩ + |11⟩) / √2
```

You measure `00` half the time and `11` half the time. You never measure `01` or `10` ideally.

This is a **Bell state**, a particular example of **entanglement**. An entangled state cannot be described as independent quantum states for its parts. [IBM's multiple-systems lesson](https://learning.quantum.ibm.com/course/basics-of-quantum-information/multiple-systems) explains this shared description.

Matching outcomes alone do not prove entanglement: ordinary paired records can match too. Distinguishing them requires additional measurement choices.

Entanglement also cannot send a message faster than light. You cannot choose your random local outcome to transmit a chosen bit.

## The measurement bottleneck

For `n` qubits, the general state description has `2ⁿ` amplitudes. Measuring all qubits once gives one `n`-bit string.

You cannot read the entire amplitude list from that result. An algorithm must arrange interference so measurements reveal useful information.

## Key takeaways

- Amplitudes determine probabilities through their squared magnitudes.
- Phase affects how amplitudes reinforce or cancel.
- Entanglement requires a shared quantum description.
- One measurement does not reveal every alternative.

**Next: [Follow a tiny quantum circuit](./02-follow-a-quantum-circuit.md)**
