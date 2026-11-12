# 5. LLM-as-Judge

Case study: an LLM that listens to recorded sales calls, scores the agent on a rubric, writes a summary and coaching points into the CRM, and extracts structured deal data.

---

## The rubric comes from the best domain expert

The first rubric was generic ("rapport", "objection handling"). It became useful after the **sales lead annotated real calls** with what the AI got wrong. Each annotation became a rule:

- **New dimensions** that matter in this business: identifying the decision maker, local and seasonal expertise, respecting the customer's timing, language register.
- **Scoring corrections:** a justified early disqualification is *good* sales work, but the rubric was scoring it 2/10. Rewarding it moved those calls to about 8.
- **Domain knowledge as hard rules:** specific product facts an expert agent must raise. Missing them caps the score.
- **Classification rules:** what counts as "prior contact", and which call type a call belongs to.

**Verify every rubric change E2E on the exact calls the expert flagged**, in a clone with CRM writes disabled.

---

## N/A must be excluded, not scored

Dimensions that didn't apply to a call were scored **3/10**, which silently pulled every good call down. Mark them `na` and **exclude them from the weighted average**. Check how your judge treats "not applicable" before trusting any average.

---

## The judge isn't reproducible

Re-rating the **same recording** with the **same prompt** at temperature 0:
- **7–12 of 15 dimensions** changed
- individual dimensions swung by as much as 7 points
- the overall score moved by **up to ~1.7 points**

Likely contributors: audio input isn't deterministic even at temperature 0, thinking tokens vary, and batch and direct APIs behave differently.

**Consequences:**
- A score is partly a property of **the run**, not only the call.
- **Before ranking agents or tracking improvement, measure the noise floor**: rate the same calls N times.
- Report scores as **bands**, average several ratings for decisions that matter, or reduce variance at the source.
- When testing a prompt change, compare **control vs. test on the same runs**. The change's real effect (here a few tenths of a point) can be much smaller than run-to-run variance.

---

## Make the judge's output verifiable

Each strength or gap starts with a `[MM:SS]` timestamp from the recording, and the summary links to the call. A rep or manager can jump to the moment and check whether the judge was right. Measured against a timestamped transcript, the timestamps landed within a few seconds.

**Trap:** adding timestamps to the *full transcript* in the output made long calls exceed the output budget. The JSON was truncated and the call scored 0. Timestamp only what a human will use.

---

## Extraction is a separate stage, with merge semantics

The judge also extracts deal data (itinerary, preferences, background) into CRM fields. Two lessons:

- **Merge, don't append.** Appending a dated block per call repeated the same information across calls. The extractor now receives the field's current value and returns one merged, deduplicated current state, written only when non-empty, so a failed extraction never wipes data.
- **Select what you're about to update.** A field missing from the read query looked empty, so every call overwrote it.

---

## Model choice

Use a **strong model** for judging and for clustering failures. Weaker models give vague, inconsistent verdicts, and the cost difference is small next to the cost of coaching people on wrong scores.
