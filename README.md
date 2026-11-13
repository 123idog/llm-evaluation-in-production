# LLM Evaluation in Production

**How to know whether an AI agent actually works, with real customers, real money and a sales team that wants proof, not demos.**

Every team deploying LLMs hits the same wall: the demo looked great, the executions are green, and nobody can say whether the bot is helping or quietly hurting. This repo covers the methods I use to answer that with evidence: what to measure, how to measure it without fooling yourself, and how to evaluate safely before and after a change goes live.

---

## Contents

| # | Chapter | What's inside |
|---|---|---|
| 1 | [What to Measure](docs/01-what-to-measure.md) | Outcomes not executions, units, denominators, autonomy ≠ quality |
| 2 | [Measurement Hygiene](docs/02-measurement-hygiene.md) | Completeness proofs, timestamp traps, survivorship, asymmetric comparisons |
| 3 | [Offline Evaluation](docs/03-offline-evaluation.md) | Replay clones, regression diffs, simulators, probe campaigns, in-sample traps |
| 4 | [Shadow Mode](docs/04-shadow-mode.md) | Compare what the bot *would* do with what humans *did*, at zero risk |
| 5 | [LLM-as-Judge](docs/05-llm-as-judge.md) | Rubrics from domain experts, N/A handling, judges that aren't reproducible |
| 6 | [Observability](docs/06-observability.md) | Tracing without latency, privacy, scores, cost that's really a floor |
| 7 | [Evaluating Third-Party APIs Before Integration](docs/07-evaluating-external-apis.md) | HTTP 200 that means failure, prices that don't reproduce |
| 8 | [Measuring Cost](docs/08-measuring-cost.md) | Cost per result not per token, tracing that can answer it, where money leaks |

---

## Seven rules

1. **A green execution proves nothing.** Measure what happened to the customer.
2. **Say "retrieved N of M" before any number.** Truncated data produces confident, wrong conclusions.
3. **Quote the denominator.** The same bot can be "74% autonomous" and "91% autonomous" at the same time.
4. **A zero or a perfect score is a bug signature.** Make every measurement catch a known-bad case first.
5. **In-sample is an upper bound.** Anything tuned on a dataset has to be re-measured on data it hasn't seen.
6. **Measure the noise before ranking.** If re-running the judge moves the score, the score is partly a property of the run.
7. **Price per result, not per token.** A cheaper model that needs longer conversations or wins fewer of them is more expensive.

---

## About

Written by **Ido Guy**, AI & Automation Engineer. I design end-to-end AI solutions for sales and operations teams, and measure what they deliver and what they cost.

[LinkedIn](https://www.linkedin.com/in/ido-guy) · Also: [WhatsApp AI Agent Design](https://github.com/123idog/whatsapp-ai-agent-design) · [n8n Production Playbook](https://github.com/123idog/n8n-production-playbook) · [Claude Code for Automation Engineers](https://github.com/123idog/claude-code-for-automation-engineers)

Tools: [llm-eval-kit](https://github.com/123idog/llm-eval-kit) (regression tests) · [llm-cost-report](https://github.com/123idog/llm-cost-report) (cost per result from Langfuse traces)

*All examples are generalised and numbers are rounded. No client data, prompts or identifiers are included.*
