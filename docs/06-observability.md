# 6. Observability

Execution logs are pruned after a few days and are built for debugging workflows, not for evaluating an agent. Add real LLM tracing, without slowing the customer down.

---

## Trace from a side branch, after the send

```
... → Send reply → Log ─┬─→ (main path ends)
                        └─→ Build Trace → POST OTLP traces → POST scores
```

- The trace branch runs **after** the reply is sent, so it adds zero customer latency.
- Every node is `onError: continue`. **Tracing must never break a turn.**
- The trace-building code returns nothing on any exception, or when the agent didn't run.
- Use deterministic IDs (a hash of `'bot-' + executionId`) so re-runs overwrite instead of duplicating, and put the execution ID in metadata to link back to the platform.

---

## Privacy by default

- **User and session IDs = a hash of the phone number.** The raw number never leaves your system.
- Mask phone-like numbers inside message text before sending.
- Decide explicitly, with the business owner, whether conversation content may go to the tracing vendor. Get the approval in writing.

---

## Tags and scores that answer real questions

| Kind | Examples |
|---|---|
| Tags | bot version, message type, `handoff`, `retry`, `gate:<name>` |
| Scores (boolean) | `handoff`, `retry_used`, `gate_fired` |
| Metadata | **prompt revision** (a constant you bump on every prompt edit), execution ID |

The prompt-revision constant is what lets you compare behaviour **before and after a prompt change**. Without it, traces from different prompts blur together.

---

## Tokens and cost: enrich nightly

The workflow platform exposes token usage per LLM call only in execution data. A **nightly enricher** reads the previous day's executions, extracts per-call token usage, timings and tool calls, and posts them as child spans.

**Cost is a floor, not a total.** The platform's token counts didn't include the model's *thinking* tokens, which can be the majority of the bill. Label the number accordingly.

---

## What the first traces showed

Even a handful of traced turns produced useful hypotheses (worth confirming on more data):
- **Customer wait time was dominated by the debounce window** (30 s), not by the model (3–15 s).
- **Input tokens grew by a few thousand per turn** within a session as history accumulated.
- **"Thanks!" turns cost a full ~30k-token call.** A cheap pre-filter for closings would save real money.

---

## Zero-touch monitoring of the live bot

Separately from tracing, a daily monitor reads recent executions **without adding anything to the live bot** and computes:
- language-integrity failures (wrong script, `???`)
- guard firing rates, retry rates and errors
- volume against a baseline
- escalation **rate** (a metric to trend, not a finding)

Watch **rates over time**. A guard rate that climbs from under 1% to over 10% in two days means the model is degrading, and you want to know before customers do.

> Platform APIs cap execution pages (e.g. 250). A busy bot exceeds that daily, so a monitor reading "the last 24 h" may really be reading the last few hours. Paginate, or label the coverage.

---

## Persist failures

Executions are pruned in days. To answer "how often does this happen?" a month later, **write every detected failure to a table** at detection time. Weekly clustering of that table turns individual incidents into patterns you can prioritise.
