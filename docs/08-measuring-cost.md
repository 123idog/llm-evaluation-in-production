# 8. Measuring Cost

"How much does the AI cost?" has three different answers, and only one of them matters to the business.

| Unit | Example | Who it's for |
|---|---|---|
| Per call | $0.0075 | The engineer picking a model |
| Per conversation | $0.10 | The finance spreadsheet |
| **Per result** | **$0.35 per qualified lead** | **The person deciding whether the bot is worth it** |

Report the last one. The first two are inputs to it.

---

## Cheaper per call can be more expensive per result

A real pattern, shown here with invented numbers ([reproducible demo](https://github.com/123idog/llm-cost-report)): a team moves its main step to a model that is 3× cheaper per token.

| | Before | After |
|---|---|---|
| Cost per model call | $0.0075 | $0.0046 (**−38%**) |
| Model calls per conversation | 13.8 | 25.1 |
| Cost per conversation | $0.103 | $0.117 (**+13%**) |
| Success rate | 30% | 19% |
| Cost per qualified lead | $0.35 | $0.61 (**+74%**) |

The cheaper model asked more clarifying questions, so conversations got longer, and it qualified fewer leads. Every per-call dashboard showed a win.

**Rule: a model change is evaluated on cost per result _and_ quality together, never on price per token.**

---

## Make the tracing able to answer the question

Cost per result needs the cost and the outcome on the same record. In Langfuse terms:

| What | How | Why |
|---|---|---|
| One conversation = one trace (or one session) | Stable ID per conversation | The unit you divide by |
| Every model call is a generation, **named by step** | `classify`, `answer`, `crm-summary` | Tells you *which part* of the pipeline costs |
| Exact model name on every generation | The real model string, not a family name | Cost is computed from it |
| Usage including **cached** input tokens | A separate cached-tokens field | Otherwise caching savings are invisible |
| The outcome as a **score on the trace** | Written later, when the CRM knows (lead qualified, ticket resolved) | Outcomes arrive hours or days after the conversation |
| Metadata: language, channel, prompt revision | Constants set by the agent | Lets you split cost by market and by version |

The outcome score is the piece most teams skip, because the outcome lives in another system. A nightly job that reads outcomes from the CRM and writes them as scores by trace ID closes the loop.

---

## Before trusting the total

1. **Count generations with no cost.** A model with no price definition is silently counted as $0. A report should say "N of M calls have no cost", never sum around them.
2. **Cost is a floor** if thinking/reasoning tokens aren't reported (see [chapter 6](06-observability.md)).
3. **Retrieved N of M.** Exports from the API are paginated. A report built on the first page describes the first page.
4. **Failed calls are billed.** Include them; they are part of what a conversation costs.

---

## Where the money usually leaks

| Leak | Signature in the traces | Fix |
|---|---|---|
| Long prompt resent uncached | A step with thousands of input tokens per call, mostly identical, 0% cached | Fixed part first and unchanged, turn on prompt caching |
| Big model for a tiny answer | A step whose output is a label or yes/no, on the most expensive model | Move to a small model, **after** an eval confirms the routing still holds |
| Failed calls + retries | ERROR-level generations followed by the same step again | Shorter timeout, fallback model, fix the cause |
| Runaway conversations | A few traces with 40+ calls | Cap turns, hand off to a human before the cap |
| Courtesy turns | "Thanks!" triggering a full agent call | A cheap pre-filter for closings |

Two of these (caching and model choice) usually account for most of the saving. Both are configuration changes, not rewrites.

---

## Look at the distribution, not the mean

The mean cost per conversation hides the tail. Report p50, p95 and **the share of spend held by the most expensive 5% of conversations**. When that share is high, the fix is in a handful of broken conversations, not in the prompt.

Split by language as well: the same conversation in Arabic or Hebrew often uses more tokens than in English, so a market can be structurally more expensive per result.

---

## Gate releases on it

Run the cost comparison on the same window before and after a change, next to the quality eval. A release that raises cost per result by more than an agreed threshold needs a reason, written down.

```bash
llm-cost compare before.json after.json --success score:qualified --max-increase 10
```
