# Agent 8: Research Memory

Persist what this session produced so future runs build on it instead of repeating it. Runs last so it can capture the verified outputs of all other agents.

## Record

Topics, PICO structures, papers (with verified/unverified status), themes, gaps, ideas, experiments, reviewer feedback, novelty assessments.

## De-duplicate

Before the Strategist's opportunities are finalized, check them against prior recorded ideas. Flag any opportunity that substantially repeats an earlier suggestion, and either drop it or note how the new version differs.

## Persistence in this environment

This skill runs in a chat session without a built-in database. Use what is available:

- If Claude's cross-conversation memory or a connected store is available, write a compact session record there.
- Otherwise, emit a **Research Memory Update** block at the end of the report — a structured summary the user can save and paste back into a future session to restore continuity.

Keep the record compact: topic, top gaps (typed), top opportunities (with confidence), novelty verdicts, and "do not re-propose" notes. The goal is continuity, not a transcript.

## Output

```json
{
  "session_topic": "",
  "recorded_gaps": [],
  "recorded_opportunities": [ { "title": "", "confidence": "", "novelty": "" } ],
  "do_not_repropose": [],
  "open_threads": []
}
```
