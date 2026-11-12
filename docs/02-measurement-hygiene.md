# 2. Measurement Hygiene

Most wrong conclusions about AI systems don't come from bad models. They come from bad measurement.

---

## Prove completeness: "retrieved N of M"

APIs truncate silently: default page sizes of 50 or 100, caps of 250, cursors that are ignored. Before any analysis:

1. State **retrieved N of M available**. If they differ, stop.
2. When you can't get M directly, **prove the candidate set is complete**. For example, contacts sorted by last activity (descending): if the last in-window contact sits at index 150 of 1,500 scanned, the set is provably complete.
3. Paginate until the cursor is null. Never trust a "latest page" scan to prove absence. Some APIs silently ignore `sortBy` and return the oldest rows.

---

## Timestamps will lie to you

- A provider returned `timestamp` in a non-ISO format. `Date.parse` gave `NaN`, **every window filter silently matched nothing**, and the result was "0 messages waiting". Use the ISO field, fall back to epoch, and assert a non-zero count on a known-busy day.
- A table filter on a timestamp-as-string column was only correct at **day** granularity. Finer comparisons returned older rows or nothing. Filter coarsely, then apply the exact window in code.
- **Pin both ends of a window** (and the candidate cutoff) to fixed timestamps when re-running for comparison, or the tail drifts.

---

## A zero or a perfect percentage is a bug signature

In one investigation three numbers were wrong in a single day: "0 waiting", a suspiciously precise burst rate, and "hundreds lost, 100%". Each came from a parsing or filtering bug.

**Rule:** before trusting a measurement, make it flag **a case you already know is bad**. A measurement that can't find a bug you know exists can't be trusted to find the ones you don't.

---

## Judge outcomes only after they can happen

"Did the customer abandon?" can't be judged ten minutes after the bot's question. Wait (12 h worked; 6 h vs. 12 h changed nothing, which is a good sign), or live conversations count as abandoned.

---

## Classify by the right text

To attribute abandonment to a question, classify the bot message by **its last question sentence only** (from the previous punctuation to the final `?`). Matching anywhere in the message mislabels it, because a framing sentence often precedes the real question.

Read state from the **bot's own wording** when the customer's reply is ambiguous. A bare "no" means different things after different questions, and the question may have been sent by a template that isn't in your log.

---

## Asymmetric comparisons

"Bot hands off in 13 minutes vs. agents' 10 hours" sounds decisive. Check what each arm measures:
- The bot arm was measured to a **fully qualified handoff**.
- The agent arm was measured only to **first touch**.

The gap is understated, not overstated, but the agent arm had a small sample. **Present it as an order of magnitude, not a precise figure**, and say why.

---

## Don't fake a missing measurement

In one bot, a gate dropped customer messages from the log once a human took over. For agent-only conversations there was **no start time**. Comparing anyway would have measured the bot from the customer's first message and the agent from the agent's first message, which isn't a comparison.

**Rule:** if the instrumentation can't support a number, report the blocker and the fix (log the muted turn), not a number.

---

## Survivorship and correlation

- Episodes that haven't reached an outcome yet are excluded from "time to outcome", so every figure is an **optimistic floor**. Say so, and check that the bias applies equally across the buckets you compare.
- Correlation isn't causation, especially when bot behaviour depends on the customer (long conversations ↔ undecided customers).

---

## Attribution of who sent what

Bot share depends on correctly classifying every outbound message as bot or human:
- Bots send through API tokens, humans through named operators, and auto-replies through a third identity.
- Unknown operator names appear. **Verify each one** before counting it on either side.
- **Validate the classifier against raw threads**, by hand, for a sample (e.g. 34 of 34 messages in 6 threads) before trusting it at scale.
