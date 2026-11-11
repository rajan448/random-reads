# 2. Follow a tiny quantum circuit

> **TL;DR:** A quantum gate changes amplitudes predictably; measurement converts the resulting state into a sampled answer.

## The pieces of a quantum program

A **quantum gate** is a controlled, reversible transformation of a quantum state. A **quantum circuit** specifies a sequence of gates and measurements.

Read the following circuit from left to right:

```text
prepare |0⟩ ── H ── H ── measure
```

`H` denotes the **Hadamard gate**, which transforms the two starting states like this:

```text
H|0⟩ = (|0⟩ + |1⟩) / √2
H|1⟩ = (|0⟩ − |1⟩) / √2
```

These rules tell you everything needed for this example. Notice the minus sign in the second rule. [IBM's quantum mechanics lesson](https://quantum.cloud.ibm.com/learning/en/courses/use-a-qc-today/quantum-mechanics-basics) gives the same gate transformations.

## First gate: create a superposition

Start with `|0⟩`. After the first `H`, the amplitudes of `|0⟩` and `|1⟩` both equal `1/√2`.

Measuring now would give either result with 50% probability. Our circuit continues without measuring.

## Second gate: combine contributions

Apply the second `H` to both terms. Each term contributes to both possible final outcomes:

| Contribution from | To final state 0 | To final state 1 |
| --- | ---: | ---: |
| First term, amplitude on 0 | +1/2 | +1/2 |
| Second term, amplitude on 1 | +1/2 | −1/2 |
| Combined amplitude | 1 | 0 |

The two contributions toward `|0⟩` reinforce. Those toward `|1⟩` cancel.

Square the final magnitudes: you measure 0 with probability one and 1 with probability zero. This calculation assumes ideal gates.

Think of combining two routes carrying waves. Their relative phases determine what reaches each exit.

## Change a phase, change the result

A **Z gate** leaves `|0⟩` unchanged and reverses the sign of the `|1⟩` amplitude. Insert it between the Hadamard gates:

```text
prepare |0⟩ ── H ── Z ── H ── measure
```

The middle state becomes `(|0⟩ − |1⟩)/√2`. The final Hadamard now cancels the contributions toward 0 and reinforces those toward 1.

You measure 1 with certainty ideally. Both middle states give 50/50 results if measured directly, yet their different phases produce different final answers. [IBM's quantum circuits lesson](https://learning.quantum.ibm.com/course/basics-of-quantum-information/quantum-circuits) explains how a Hadamard distinguishes these states.

## Repeat the experiment

A **shot** is one execution with fresh preparation and measurement. Repeated shots estimate outcome probabilities.

For `H → measure`, 100 shots average about 50 zeros, with random variation. For ideal `H → H → measure`, every shot gives zero.

Try predicting `H → Z → Z → H`. Two sign reversals undo each other, so you get zero.

These circuits teach the mechanism. Their small size makes them easy to simulate on an ordinary computer.

## Key takeaways

- Gates transform amplitudes before measurement.
- Contributions add before their magnitudes are squared.
- Phase differences can become different observable answers.
- Repeated shots reveal a circuit's output distribution.

**Next: [Useful problems and physical limits](./03-useful-problems-and-limits.md)**
