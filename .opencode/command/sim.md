---
description: Query or maintain Self-Improvement Memory (SIM) for harness changes and experiments
---

Use the versioned SIM store at `.opencode/sim-memory.json`. It is shared across sessions and model/provider switches.

Requested operation: $ARGUMENTS

## Query behavior

If the request is a query (including an empty or vague request):

1. Read the entire SIM store.
2. Match records by task domain, benchmark/workflow, model/provider, root-cause hypothesis/evidence, affected concepts, tags, and status. Search historical records too; do not filter out reverted or superseded attempts.
3. Return matching IDs, status, provenance, root cause, rationale, predicted outcome, observed outcome, and why each match matters.
4. Surface every matching `active_guardrail` prominently. Explain conflicts between a proposed change and existing guardrails before recommending edits.
5. If there are no matches, say so. Do not invent a record.

## Add behavior

When asked to add/record/log an experiment:

1. Gather facts from the current task, tool output, test output, and Git diff. Do not infer missing results.
2. Require a prediction that was written before testing. If it was not captured beforehand, state that it was reconstructed after the fact and do not present it as a genuine pre-test prediction.
3. Append one record to the `records` array. Never replace the file with a single record or rewrite unrelated history.
4. Use this shape (all keys are required; unknown provenance values may be null):

```json
{
  "id": "sim-YYYYMMDD-short-description-01",
  "timestamp": "YYYY-MM-DDTHH:mm:ss+01:00",
  "provenance": {
    "policy_model": "model name or null",
    "policy_provider": "provider name or null",
    "optimizer_model": "model name or null",
    "optimizer_provider": "provider name or null",
    "task_domain": "harness/tooling/verification/etc.",
    "benchmark_or_workflow": "specific benchmark, test suite, or workflow",
    "iteration_round": 1
  },
  "root_cause": {
    "hypothesis": "underlying cause, not just the symptom",
    "evidence": ["observable evidence"],
    "confidence": "low | medium | high"
  },
  "rationale": "why the proposed change addresses the root cause",
  "predicted_outcome": {
    "expected_behavior": "what should change",
    "verification_criteria": ["objective pass/fail criteria"]
  },
  "observed_outcome": {
    "result": "what actually happened",
    "verification": ["commands, checks, and their actual results"],
    "regressions": [],
    "deadlocks_or_timeouts": [],
    "side_effects": []
  },
  "status": "verified"
}
```

5. Choose status precisely:
   - `verified`: criteria were executed and the intended behavior passed.
   - `reverted`: the attempted change was rolled back.
   - `superseded`: a later record replaces this conclusion.
   - `active_guardrail`: a tested invariant/protection that future changes must preserve.
6. Validate JSON syntax and required keys, ensure IDs are unique, and confirm the new record is present after writing.

## Update behavior

When asked to update a record, preserve its historical facts. Prefer appending a new record and marking the earlier one `superseded` or `reverted`; do not silently rewrite predicted or observed outcomes. Only correct a factual transcription error in place, and explain the correction.

## Query examples

- `/sim query shell timeout and background jobs`
- `/sim query permission interlocks and verification gates`
- `/sim query model=gemma provider=lm-studio domain=tool-execution`
- `/sim add the completed experiment from this session`
- `/sim update sim-20261127-shell-timeout-01 as superseded by ...`

When editing JSON, preserve the complete existing document and use an atomic write where available. Do not claim persistence or verification unless it succeeded.
