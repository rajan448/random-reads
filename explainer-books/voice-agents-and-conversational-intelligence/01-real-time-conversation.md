# 1. Make the conversation work in real time

> **TL;DR:** Natural interaction requires good turn decisions, coordinated streaming, and cancellation across the entire audio path.

## Lesson 1: Choose your architecture for the failure you must debug

A **cascaded pipeline** chains specialized components:

```text
microphone → STT → LLM → TTS → playback
                  ↕
             application tools
```

**STT**, or speech-to-text, transcribes speech; **ASR**, automatic speech recognition, means the same task here. An **LLM**, or large language model, generates language and proposed actions. **TTS**, text-to-speech, produces spoken audio.

A **speech-to-speech model** accepts audio and generates audio more directly. It can preserve vocal cues that a text transcript loses. A cascade exposes intermediate text and separate components, making some debugging and substitutions easier. Neither architecture automatically solves action reliability or turn-taking. [Pipeline comparison](https://docs.livekit.io/agents/models/pipelines/).

Choose using representative calls. A scripted greeting does not exercise corrections, overlap, or tool failures.

## Lesson 2: Silence does not necessarily finish a turn

**VAD**, voice activity detection, estimates whether audio contains speech. **Endpointing** decides when to accept a user's turn as complete.

Consider: “My number is nine eight…” followed by a pause. Speech has stopped temporarily, but the user may still be collecting their thoughts.

Use speech activity, linguistic context, and task context together. A number-collection task may need different behavior from a yes/no question. Aggressive endpointing reduces waiting but increases premature replies. Conservative endpointing avoids some cutoffs but creates silence. [Turn-management mechanics](https://docs.livekit.io/agents/logic/turns/).

Do not confuse a final STT segment with a complete conversational turn. Segment boundaries belong to the recognizer; conversational completion belongs to your interaction policy.

## Lesson 3: Measure the response the caller experiences

A **trace** records related, timestamped events. Trace speech ending, turn acceptance, transcript readiness, model output, TTS output, and playback.

For an illustrative cascade, measure cumulative times after the user's last speech:

| Event | Time |
| --- | ---: |
| Turn accepted | 400 ms |
| Transcript ready | 450 ms |
| First usable model text | 700 ms |
| First synthesized audio available | 900 ms |
| First audio played | 980 ms |

These are invented teaching numbers, not service benchmarks. The caller waits 980 ms; the model-text interval accounts for only part of that wait.

**Streaming** processes and delivers chunks before the complete output is ready. Stages can overlap, so do not blindly add their independently reported durations. Server audio delivery also differs from device playback. [Session timing reference](https://docs.livekit.io/reference/agents/session-traces/).

Measure median, or **p50**, and **p95**, the latency at the 95th percentile. Slow turns can disappear inside an acceptable average.

## Lesson 4: Interruption requires coordinated cancellation

**Barge-in** is a user speaking to take the turn while the agent speaks. A **backchannel**, such as “mm-hmm,” may simply signal attention. Treating both identically produces unnecessary stops. [Interruption distinction](https://docs.livekit.io/agents/logic/turns/adaptive-interruption-handling/).

When yielding, stop obsolete generation, speech synthesis, and queued playback. Clear the **playback buffer**, audio waiting to be played. Use a **turn revision**, an identifier that changes when the accepted turn changes, to reject late output from superseded work.

Stopping only the model leaves already synthesized speech talking over the user. Conversely, cancelling speech does not undo a completed backend action. [Pipeline cancellation behavior](https://docs.pipecat.ai/pipecat/fundamentals/interruptions).

Think of a restaurant changing an order: stopping the waiter does not automatically stop the kitchen.

## Key takeaways

- Select architecture using realistic failure scenarios.
- Separate speech detection from turn completion.
- Measure until audio plays, with explicit timing boundaries.
- Cancel stale output across generation, synthesis, and playback.

**Next: [Keep state and actions reliable](./02-state-and-action-reliability.md)**
