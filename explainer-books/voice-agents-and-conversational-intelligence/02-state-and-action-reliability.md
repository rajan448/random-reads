# 2. Keep state and actions reliable

> **TL;DR:** Treat speech as revisable input, maintain explicit application state, and distinguish attempted actions from confirmed outcomes.

## Lesson 5: A transcript is an observation, not ground truth

Streaming recognizers emit **partial transcripts**, hypotheses that can change as speech continues. A **final transcript** completes a recognized segment; it can still contain recognition errors. [Streaming transcription behavior](https://docs.aws.amazon.com/en_en/transcribe/latest/dg/streaming-partial-results.html).

Suppose the caller says: “Book Tuesday… no, Wednesday.” Acting on the earliest recognizable date creates the wrong booking.

Use partial text for captions or reversible preparation. Wait for sufficient turn evidence before committing consequential changes. Read back critical dates, identifiers, or amounts when ambiguity would affect the outcome.

A confidence score is a model's estimate, not a substitute for checking the requested details. [Recognition confidence](https://docs.cloud.google.com/speech-to-text/docs/overview).

## Lesson 6: Separate conversation history from application state

**Structured state** stores explicit fields such as requested date, confirmed date, and booking status. A **state machine** permits defined transitions between states.

For a booking, one possible design is:

```text
collect details → clarify ambiguity → confirm details
                → submit request → verify result → report outcome
```

Let the model interpret language and propose the next step. Application code checks permissions, required fields, and allowed transitions. A caller's request cannot rewrite those checks.

Track distinct records:

| Record | What it establishes |
| --- | --- |
| Transcript | What recognition reported the user said. |
| Generated reply | What the model planned to communicate. |
| Played reply | What playback telemetry indicates was delivered. |
| Action record | What the backend reports actually happened. |

Playback does not prove the user heard or understood the message.

An interrupted reply should not remain in history as though its entire generated text was spoken. Preserve the delivered portion using the best available playback alignment. Pipecat documents this distinction in its [conversation-context handling](https://docs.pipecat.ai/pipecat/fundamentals/interruptions).

## Lesson 7: Make retries preserve the intended action

A **tool** is an application function the agent can request, such as checking availability or creating a booking. A timeout means the result is unknown; it does not prove failure.

**Idempotency** makes repeating the same intended operation avoid duplicate effects. Use an **idempotency key**, a stable identifier for that operation, when the backend supports it.

If a booking succeeds but its response is lost, retry with the same operation key. Have the service return the recorded outcome instead of creating another booking. A changed date represents changed intent; do not silently reuse the old key with new parameters. [Safe retry design](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/).

Do not promise success before checking the backend response. If the outcome remains uncertain, say so and reconcile it through a status lookup or human handoff.

## Lesson 8: Make recovery part of the dialogue

Short speech turns make corrections easier. Ask one focused question, preserve confirmed details, and repair the specific uncertainty.

For “Did you say fifteen or fifty?”, request the amount again instead of restarting the entire conversation.

A **handoff** transfers responsibility to a person or another workflow. Include the caller's goal, verified details, unresolved questions, and action status. Preserve pending-operation identifiers so another worker does not repeat an uncertain action.

Prefer a specific status update over repetitive filler. “I'm checking availability” describes ongoing work; “You're booked” asserts an outcome.

These state and recovery patterns are design recommendations, not guarantees supplied by a speech model.

## Key takeaways

- Recognition output can change or be wrong.
- Store confirmed facts and action status explicitly.
- An interrupted sentence and an interrupted action need different handling.
- Retry with stable operation identity; reconcile uncertain outcomes.
- Give handoffs enough context to continue correctly.

**Next: [Turn conversations into trustworthy intelligence](./03-intelligence-and-evaluation.md)**
