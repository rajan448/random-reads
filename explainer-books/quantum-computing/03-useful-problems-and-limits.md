# 3. Useful problems and physical limits

> **TL;DR:** Quantum computing helps when a problem supports an efficient quantum algorithm and the hardware can execute it reliably.

## What the machine physically does

A qubit needs a controllable physical system with two usable quantum states. Implementations include trapped atoms or ions (electrically charged atoms) and specially engineered electrical circuits.

Control signals implement gates. Detectors turn measurements into ordinary digital results. A conventional computer prepares instructions and processes those results.

The equipment depends on the implementation. Superconducting circuits, which carry electricity without resistance, require extreme cooling. Other approaches have different requirements. [NIST's explanation](https://www.nist.gov/quantum-information-science/quantum-computing-explained) describes these physical approaches and their challenges.

## Where an advantage can come from

An **algorithm** is a procedure for solving a problem. A **quantum advantage** means outperforming a relevant classical approach on a specified task and measure.

Different tasks offer different possibilities:

| Task | Why quantum computing enters the picture |
| --- | --- |
| Simulating molecules | Molecules obey quantum rules; quantum hardware can represent their behavior directly. |
| Factoring integers | Shor's algorithm efficiently exploits mathematical repetition on a sufficiently capable quantum computer. |
| Unstructured search | Grover's algorithm reduces the number of checks needed under a specified access model. |

Factoring means splitting an integer into prime factors: for example, `15 = 3 × 5`. [Shor's original paper](https://arxiv.org/abs/quant-ph/9508027) establishes an efficient quantum factoring algorithm. This matters for encryption schemes whose security relies on factoring being difficult.

For a search with one marked answer among a million candidates, classical checking takes roughly a million checks in scale. Quantum search needs roughly a thousand in scale, under its assumptions. These are growth estimates, not exact operation counts. [Grover's original paper](https://arxiv.org/abs/quant-ph/9605043) establishes this square-root improvement.

The checking operation must itself have an efficient quantum implementation. Loading data, controlling hardware, and correcting errors can change the overall cost.

## Why reliability is difficult

**Noise** means unwanted disturbances or imperfect control. **Decoherence** is the loss of quantum coherence through interaction with the environment; it damages the phase relationships computation needs.

**Quantum error correction** protects encoded information using extra qubits and measurements designed to reveal errors without reading the encoded answer. [IBM's error-correction introduction](https://quantum.cloud.ibm.com/learning/en/courses/foundations-of-quantum-error-correction/correcting-quantum-errors/introduction) explains the reliability challenge.

A **physical qubit** is an individual hardware carrier. A **logical qubit** is an encoded qubit protected using multiple physical qubits. Their numbers therefore measure different resources. [IBM's detailed error-correction lesson](https://quantum.cloud.ibm.com/learning/en/courses/tools-for-quantum-advantage/error-correction-ii) explains the resource tradeoffs.

## How to evaluate a claim

Ask what problem was solved, what classical method was compared, and whether the comparison includes the full cost.

As of November 2026, error correction remains an active engineering and research effort. Check current evidence before relying on hardware forecasts. Algorithmic promise alone does not establish a useful deployed system.

## Key takeaways

- Useful speedups depend on a specific algorithm and problem.
- A reduced check count does not guarantee a faster complete application.
- Noise and error correction shape the hardware requirements.
- Qubit counts need context about reliability and encoding.

**Where to go deeper:** Recreate the previous chapter's three circuits on paper. Then follow [IBM's circuit-running lesson](https://quantum.cloud.ibm.com/learning/en/courses/quantum-computing-in-practice/running-quantum-circuits) to connect diagrams with execution.
