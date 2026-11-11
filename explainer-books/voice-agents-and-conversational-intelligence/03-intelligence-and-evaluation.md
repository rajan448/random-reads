# 3. Turn conversations into trustworthy intelligence

> **TL;DR:** Preserve evidence, distinguish statements from outcomes, and evaluate complete conversations as well as individual models.

## Lesson 9: Preserve who said what and when

**Conversational intelligence** extracts useful information from conversations: topics, commitments, summaries, and outcomes. It can operate during a call or afterward.

**Speaker diarization** groups speech by speaker. It answers “which speaker spoke when,” not automatically “what is this person's verified identity?” Numbered speaker labels are not authenticated names. [Diarization documentation](https://docs.cloud.google.com/speech-to-text/docs/multiple-voices).

For every extracted claim, preserve **provenance**, the evidence supporting it. A useful record includes the conversation identifier, timestamp range, speaker label, source words, and analysis version.

Keep speaker and transcript uncertainty visible. A fluent summary cannot repair incorrectly attributed speech without additional evidence.

## Lesson 10: Extract facts before writing polished summaries

Use a **schema**, a defined set of fields, to separate requests, commitments, completed actions, and unresolved issues.

Consider this invented exchange:

```text
00:14 Caller: "I need to change the delivery date."
00:22 Agent:  "I'll check whether Wednesday is available."
00:35 Tool:   availability lookup times out
```

Supported extraction:

```text
request: change delivery date
proposed_date: Wednesday
availability: unknown
change_status: not confirmed
```

“Delivery moved to Wednesday” is unsupported. A promise to check is not a completed change. Summarize from the supported fields and cite their evidence.

Think of a summary as a report with receipts. For a completed backend action, the receipt is the action record, not merely a sentence in the transcript.

Keep **sentiment**, an inferred attitude label, separate from explicit statements and verified outcomes. Sarcasm, accent, and context can complicate interpretation. This is a design reason to expose uncertainty rather than present emotional labels as facts.

## Lesson 11: Measure the errors that change outcomes

**WER**, word error rate, compares recognized words against a reference transcript:

```text
WER = (substitutions + deletions + insertions) / reference word count
```

For example, one substituted word in a 100-word reference yields 1% WER. If that word changes “fifteen” to “fifty,” the task may still fail. [Speech-accuracy measurement](https://docs.cloud.google.com/speech-to-text/docs/v1/speech-accuracy).

Use several measures:

| Layer | Example measurement |
| --- | --- |
| Recognition | WER and correct critical dates, amounts, names, and negations. |
| Conversation | Premature replies, missed interruptions, repair success, and response latency. |
| Actions | Verified task completion, duplicates, and unresolved outcomes. |
| Intelligence | Unsupported claims, missing required facts, and incorrect speaker attribution. |

Do not equate a pleasant call with a correct outcome. Conversely, a successful booking does not prove its summary is accurate.

## Lesson 12: Test the system through the channel users experience

A **regression set** is a repeatable collection of cases used to detect deterioration after changes.

Include corrections, long pauses, short acknowledgments, overlapping speech, noise, language changes, and backend timeouts. Compare results across relevant call conditions instead of relying on one aggregate score.

Text tests can check decisions and tool arguments. Audio tests exercise recognition, timing, pronunciation, and interruption handling. Simulations help reproduce scenarios but do not replace reviewing representative real calls. [Text and audio simulation testing](https://docs.livekit.io/testing/simulations/).

Change one major component at a time and inspect failures. Version the model settings, prompts, turn policy, and extraction schema together with evaluation results.

Retain the evidence needed for debugging under defined access and retention rules. Remove unnecessary sensitive fields from routine analytics views.

## Key takeaways

- Preserve evidence and speaker uncertainty alongside extracted claims.
- Distinguish requested, promised, and completed actions.
- Combine component metrics with verified task outcomes.
- Test audio behavior and recovery paths, not only text responses.

**Where to go deeper:** Build a small appointment-booking evaluation set. For every case, specify the expected conversation behavior, action record, and final summary.
