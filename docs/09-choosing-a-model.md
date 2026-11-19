# 9. Choosing a Model: Why Single-Turn Evals Lie

A model change is the riskiest edit you can make to an AI agent. Every prompt was tuned against the old model, and the new one fails in different places. This chapter is the process I used when a provider retired the model our agents ran on, and what each step caught that the previous one missed.

Model names are replaced with letters. The lessons don't depend on which vendor it was.

---

## Deprecations are silent

The retirement notice said that after the cutoff date, requests to the old model name would be **routed automatically** to a newer model. No error, no failed executions. On that date every agent would start running on a model nobody had tested, and the old model, the only baseline you could compare against, would no longer exist.

Two consequences:

1. **The comparison has to run before the cutoff**, while the baseline still answers.
2. One of the "upgrades" was in fact a **downgrade** to a lighter model. Pin the model you choose explicitly; don't let a default decide.

---

## Step 1: Inventory what actually runs

The old model name appeared in **150 workflows**. Only **31 were active**, and a handful of them carried almost all the traffic.

| Cluster | Examples | How to treat it |
|---|---|---|
| Agent "brain" | The main reply step of each chatbot | Full evaluation, below |
| Media | Reading images, audio, documents sent by customers | Spot-check with real files |
| Transcription | Sales-call audio | Compare on real calls, per language |
| Low stakes | Review replies, summaries | Switch, then watch |
| Test and dead | Replay clones, debug copies | **Deactivate, don't migrate** |

Migrating dead workflows is wasted work. The inventory turned a 150-item problem into a 5-item one.

---

## Step 2: Single-turn comparison on real data

Real customer turns, replayed against six candidate models, scored by a blind LLM judge that saw each reply without knowing which model wrote it.

This produced a clear ranking and a recommendation. **It was wrong for one of the two agents.**

---

## Step 3: Multi-turn replay reversed it

Next, 50 real conversations (up to six turns each), where **each model continued the conversation with its own previous replies** instead of the human agent's.

The model that had placed second in the single-turn test for one agent dropped to last:

| Failure, across 25 conversations | Count |
|---|---|
| Asked a question the customer had already answered | 14 |
| Ignored the customer's message and continued its own script | 12 |
| Missed a handoff to a human | 9 |
| Contradicted something it said earlier | 9 |

None of these can appear in a single-turn test. Each one needs the model to read its own earlier output and stay consistent with it. A single-turn eval with a recorded history quietly tests a different, easier task: *continue a conversation a competent human has been running*.

**Rule: for anything conversational, choose models on multi-turn replay with the model's own history.** Single-turn results are a first filter, not a decision. The [multi-turn mode in llm-eval-kit](https://github.com/123idog/llm-eval-kit#5-multi-turn-replay-why-single-turn-evals-lie) reproduces this with invented data.

---

## Step 4: The judge and the human disagreed

The person who knew the agents best read about ten conversations by hand and preferred the incumbent model. The blind judge ranked the incumbent near the bottom, with the most hard flags.

Both can be wrong, so neither decides alone:

- **Check the judge's context.** Several "wrong price" flags came from a reference catalog that lacked data the agent fetched live through a tool. The judge flagged correct answers. Give the judge the same facts the agent had.
- **Check the human's sample.** Ten conversations read with a known model name is not blind and not large.
- **The decisive step is a blind human review**: the same conversations side by side, model names hidden, scored by the people who own the outcome.

---

## Step 5: Constraints that scores don't show

| Constraint | What happened |
|---|---|
| **Latency** | The best-scoring model took a median of ~40 s per turn and a p95 over 3 minutes with tools. Unusable in a chat, whatever its score. |
| **Output format** | One model drifted on a status line that a parser downstream depends on (13 times). Another returned invalid JSON in 10% of turns. A pretty reply that breaks the pipeline is a failed reply. |
| **Language** | On Arabic call transcription, the incumbent wrote Arabic words in Hebrew script and got stuck in a loop. The best model for Hebrew was not automatically the best for Arabic. |
| **Cost per result** | Compare the cost of a *successful conversation*, not the price per token ([chapter 8](08-measuring-cost.md)). |

---

## Decide per step, not per company

The outcome was not "we're moving to model X". It was a table: one model for the main agent, another for transcription, an optional stronger one for Arabic audio only, with prompt hardening where the chosen model over-promised. Each row had its own evidence.

---

## Checklist

1. Read the deprecation notice for **silent reroutes and downgrades**. Note the date the baseline disappears.
2. Inventory active usage. Deactivate the dead.
3. Single-turn comparison on real data, blind judge: a first filter.
4. **Multi-turn replay with the model's own history**: the decision.
5. Judge with the same context the agent had. Settle judge vs human disagreements with a blind human review.
6. Check latency (p95), format compliance and each language separately.
7. Decide per pipeline step. Pin model names explicitly.
8. After switching, compare live metrics before vs after on the same window ([chapter 2](02-measurement-hygiene.md)).
