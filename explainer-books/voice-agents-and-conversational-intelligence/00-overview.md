# Lessons from Voice Agents and Conversational Intelligence — Explainer Book

> **You'll come away knowing:** How to design responsive voice interactions, keep actions reliable, and produce evidence-backed conversation analysis.
> **For:** Engineers and product builders; no speech-processing background needed · **Total reading time:** ~11 minutes

## Why this matters

A voice agent listens, acts, and speaks while callers interrupt or revise requests. Conversational intelligence turns interactions into useful, supportable information.

You must coordinate audio, timing, application state, and evidence.

## The 60-second version

- Separate detecting speech from deciding that a person has finished their turn.
- Trace the complete path to played audio; model response time explains only part of the wait.
- Cancel stale output throughout the pipeline when the user interrupts or changes intent.
- Keep recognized speech, generated replies, played speech, and backend outcomes as distinct records.
- Extract claims with supporting evidence, then evaluate conversation quality and verified task completion separately.

## The two connected systems

```text
LIVE INTERACTION
audio → recognition → dialogue + application state → tools / reply → audio
  │                              │                       │
  └──────── timestamps, transcript, playback, action events ┘
                                 ↓
CONVERSATION ANALYSIS
speaker attribution → evidence-linked facts → summaries / quality measures
```

## Key terms

| Term | Plain-English definition |
| --- | --- |
| Voice agent | Software that interacts through speech and may perform actions. |
| Conversational intelligence | Extracting useful facts, summaries, and measures from conversations. |
| Cascaded pipeline | Specialized speech and language components chained together. |
| STT / ASR | Speech-to-text / automatic speech recognition. |
| LLM / TTS | Large language model / text-to-speech. |
| Speech-to-speech model | A model processing audio input and generating audio output. |
| VAD | Voice activity detection: estimating whether speech is present. |
| Endpointing | Deciding when to accept a user's turn as complete. |
| Streaming | Process or deliver chunks before the complete result is ready. |
| Trace | Related, timestamped events describing an interaction. |
| p50 / p95 | Median / 95th-percentile value of a measured distribution. |
| Barge-in / backchannel | Taking the turn during agent speech / a listening acknowledgment. |
| Playback buffer | Audio queued to be played. |
| Turn revision | An identifier distinguishing the current accepted turn from obsolete work. |
| Partial / final transcript | Revisable recognition output / completed recognition for a segment. |
| Structured state | Explicit fields tracking details and workflow status. |
| State machine | Defined states and allowed transitions between them. |
| Tool | An application function an agent can request. |
| Idempotency / idempotency key | Avoid duplicate effects on retry / a stable operation identifier. |
| Handoff | Transfer responsibility with the information needed to continue. |
| Speaker diarization | Grouping speech segments by speaker. |
| Provenance | Evidence showing where a claim came from. |
| Schema | Defined fields and formats for stored information. |
| Sentiment | An inferred attitude label, distinct from a verified fact. |
| WER | Word error rate: recognition edits divided by reference words. |
| Regression set | Repeatable cases for checking whether changes cause deterioration. |

## Chapters

1. [01 — Make the conversation work in real time](./01-real-time-conversation.md) — Architecture, turn-taking, latency, and interruption lessons.
2. [02 — Keep state and actions reliable](./02-state-and-action-reliability.md) — Revisions, retries, uncertain results, and recovery.
3. [03 — Turn conversations into trustworthy intelligence](./03-intelligence-and-evaluation.md) — Speaker attribution, evidence, summaries, and evaluation.

Read the overview, then chapters 1–3; discuss the examples aloud.

## Scope and assumptions

This is a synthesis of documented mechanisms and engineering recommendations, not a firsthand project retrospective. All call scenarios and timing numbers are illustrative.

The guide covers live voice agents and conversation analysis. It omits model training, vendor selection, telephony setup, and legal requirements.

As of November 2026, SDK features and defaults evolve quickly. Follow the linked primary documentation when implementing a particular runtime; validate recommendations against your own calls.
