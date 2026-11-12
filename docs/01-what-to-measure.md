# 1. What to Measure

## Outcomes, not executions

An execution log answers "did the code run?". The business asks different questions:
- Did the customer get an answer, and how fast?
- Was the lead qualified **correctly**?
- Did the conversation end in a sale, a handoff, or silence?
- How much human work did the bot remove, and how much did it add?

Pick the **unit** before the metric:

| Unit | Good for | Trap |
|---|---|---|
| **Message** | Workload share ("who sends the outbound messages") | Bots send more, shorter messages, which inflates their share |
| **Turn** | Guard rates, latency, cost | Says nothing about outcomes |
| **Conversation** | Autonomy, abandonment, handoff rate | Needs a session definition (e.g. 24 h of silence = new session) |
| **Episode** (first message → outcome) | Time to payment, time to handoff | Repeat buyers count more than once, so say so |

---

## Quote the denominator

From one real 24-hour measurement:

- Conversations closed by the bot alone, out of **all** conversations: **~74%**
- Conversations closed by the bot alone, out of conversations handled by **one side only**: **~91%**

Both are true. Whoever reads "91%" assumes the first definition. **Always state which one you mean.**

---

## Autonomy isn't quality

"No human intervened" proves the bot **finished** the conversation. It doesn't prove the bot finished it **correctly**: right qualification, right status, right handoff. Track them as separate metrics:

| Metric | Question |
|---|---|
| Autonomy | Did a human have to step in? |
| Correctness | Was the CRM outcome what the methodology says? (sampled and reviewed) |
| Speed | Time to first reply, time to qualified handoff |
| Abandonment | Where in the flow do customers stop answering? |
| Business outcome | Time to payment, conversion of bot-handled vs. human-handled leads |

---

## Beliefs vs. data

The team believed a bot was handling "about 80% of conversations, with agents doing only availability and payment". Measured over three weeks:

- The bot sent **about a fifth** of outbound messages.
- Only a few percent of conversations were bot-only. Most were mixed, and a third never touched the bot.
- Half of agent messages were substantive work (discovery, problem-solving, selling), not just closing.
- **What was true:** the bot opened most mixed conversations and was the only coverage on the weekly day off.

That reframed the roadmap from "the bot is nearly done" to "the bot is a good opener, and here's the work that's still human." **Measure claims before building on them.**

---

## Find the real drop-off

An abandonment study measured, per question, how often customers stopped answering:
- The question everyone feared (budget) had **below-average** drop-off.
- The opening question had the **highest**.
- The biggest driver wasn't any question. It was a **customer segment**: leads without a firm plan stalled several times more often, regardless of what the bot asked.

The fix for that isn't a better prompt. It's a follow-up policy for that segment. You only find this by measuring per question **and** per segment.

---

## Business outcome: time to payment

For one bot: time from the customer's first message to a payment link, bucketed by how many turns the bot held the conversation before handoff:

| Bot turns before handoff | Typical time to payment link |
|---|---|
| 1–2 | ~1 day |
| 3–5 | ~3 days |
| 6–10 | ~4 days |
| 11+ | ~8 days |

Monotonic and stable across windows, but **correlation, not causation**: long bot conversations probably indicate less decided customers. Present it with that caveat, and use it to ask the right question ("should the bot hand off earlier?") rather than to claim an answer.
