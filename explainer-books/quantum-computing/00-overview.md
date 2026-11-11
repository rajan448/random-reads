# Quantum Computing — Explainer Book

> **You'll come away knowing:** How qubits, interference, gates, and measurement fit together, and why useful speedups require carefully designed algorithms.
> **For:** A curious beginner · **Total reading time:** ~9 minutes

## Why this matters

Quantum computing uses quantum physics to process information. Understanding its mechanism helps you assess claims about scientific computing, encryption, and future hardware.

Your starting point is familiar: a computer prepares information, transforms it, and reads a result. The new ingredient is how quantum states transform before you read them.

## The 60-second version

- An ordinary bit stores 0 or 1. A qubit can have a superposition of the corresponding quantum states.
- A quantum state assigns amplitudes to possible measurement outcomes. Squared amplitude magnitudes give their probabilities.
- Gates change those amplitudes. Interference lets contributions reinforce or cancel, changing which results you are likely to observe.
- Multiple qubits can share an entangled state. Measurement still returns a limited classical result each run.
- An algorithm must connect the problem's structure to useful measurement outcomes. You cannot simply read every possible answer.

For the underlying state-and-measurement framework, see [IBM's introduction to quantum information](https://quantum.cloud.ibm.com/learning/en/courses/basics-of-quantum-information/single-systems/introduction).

## Key terms

| Term | Plain-English definition |
| --- | --- |
| Bit | An ordinary binary information unit: 0 or 1. |
| Classical | Using ordinary, nonquantum information processing. |
| Qubit | An information carrier with two quantum reference states. |
| Superposition | A quantum combination of reference states. |
| Amplitude | A mathematical weight whose squared magnitude gives a probability. |
| Phase | An amplitude's orientation, which affects how contributions combine. |
| Interference | Reinforcement or cancellation when amplitudes combine. |
| Entanglement | A shared state that cannot be separated into independent states of its parts. |
| Measurement | A physical process producing an observable result. |
| Gate / circuit | A reversible state transformation / a sequence of gates and measurements. |
| Hadamard / Z | Gates that mix amplitudes / reverse the sign of the `1` amplitude. |
| Bell state | One of four standard entangled states of two qubits. |
| Shot | One circuit execution with fresh preparation and measurement. |
| Algorithm | A procedure for solving a problem. |
| Quantum advantage | Better performance than a relevant classical approach on a specified task. |
| Noise / decoherence | Unwanted disturbances / loss of quantum coherence through environmental interaction. |
| Quantum error correction | Encoding and checks that protect quantum information from errors. |
| Physical / logical qubit | An individual hardware carrier / an encoded, protected qubit. |
| Ion / superconductor | An electrically charged atom / a material carrying electricity without resistance. |

## Chapters

1. [01 — Qubits, waves, and shared states](./01-qubits-and-interference.md) — Build the concepts behind quantum computation.
2. [02 — Follow a tiny quantum circuit](./02-follow-a-quantum-circuit.md) — Calculate how interference produces a certain answer.
3. [03 — Useful problems and physical limits](./03-useful-problems-and-limits.md) — Connect the mechanism to algorithms and hardware.

Read this overview, then chapters 1–3. Allow extra time to work through the circuit table.

## Scope and assumptions

You need no physics background or programming experience. The examples use fractions, square roots, and positive or negative amplitudes. General amplitudes can be complex numbers, which have real and imaginary parts; you do not need their arithmetic here.

The book covers computation built from quantum gates. It skips alternative computing models, full algorithm proofs, and hardware construction details.

Foundations are stable. Hardware status and availability change; check for updates beyond the November 2026 context in chapter 3.
