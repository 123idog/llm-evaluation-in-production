# 3. Offline Evaluation

Before a change touches customers, prove it on real past data. These are the harnesses that earned their place.

---

## Replay clone: regression on real turns

A copy of the production workflow that accepts **historical turns** (customer message + the history as it was at the time) and returns the reply **without sending anything**.

- Run the new version over a fixed set of real turns (60–70 is enough for most changes).
- **Diff** the replies against the current version. The result should be *only the intended differences*.
- Freeze the clock to the original execution's start time, or time-dependent logic (opening hours, follow-up timing) diverges.
- Restrict the clone to a reserved fake-number range so it can never message a real customer.

> ⚠️ Keep the clone **in sync with live after every deploy.** A stale clone gives you a regression report about code that no longer exists.

---

## Assertion suites for guards

For any deterministic guard (price verification, scaffolding detection, classification veto), write assertions from **real failures plus real good replies**:

- Every known failure must trigger with the right reason.
- Every normal reply shape must pass through untouched: prices, links, handoff tags, emoji-only, long priced answers.

Then **replay the whole log** through the guard and look at what it *would* have caught. In one case the retry gate would have fired on ~1.4% of historical replies, which were exactly the turns that had been lost to a handoff.

---

## Forced-branch clone

Some branches only run when the model misbehaves (a retry after a degenerate output). You can't trigger them on demand, so **force them**: a throwaway clone where the branch condition is always true. This is how a retry that **never saw the customer's question** (it read the wrong field) was caught before it mattered.

---

## Customer simulator

To compare a new bot with approved behaviour, replay **whole conversations**: the real customer's messages fed turn by turn, or a model playing the customer from a scenario card. Diff the outcome (status, handoff, fields collected), not the wording.

---

## Probe campaigns

For accuracy-critical behaviour (prices), run a **campaign** against the live bot:
- Dozens of fake identities, each asking point-blank about **one product**, so attribution is exact.
- Verify each answer against the **source document**, not only against the database extracted from it. One campaign confirmed *source PDF = database = delivered message* on the products checked.

**What campaigns find besides accuracy:** every probe that failed to return a price failed the same way. The bot asked "which location?" even for single-location products, because a tool parameter was declared **required** while the description called it optional. The probes exposed a schema bug no unit test would have found.

---

## In-sample is an upper bound

A pricing rule set derived from agents' past quotes scored **~92% exact** on that same corpus. On a smaller, more recent slice it scored **~83%**. The first number is an upper bound because the rules were built from that data.

**Rule:** report in-sample numbers as upper bounds, and get an out-of-sample number (shadow mode, see [chapter 4](04-shadow-mode.md)) before claiming accuracy.

---

## Corpus first, then rules

For classifiers and regex lexicons:
1. Collect **every real positive** (e.g. all genuine job inquiries) and a set of hard negatives.
2. Write the rule.
3. Run it against both, and re-run after **every** edit.

The corpus is what showed that a "simple" veto would have blocked real candidates, and that a widened lexicon matched ordinary words in another context.

---

## Clones with writes disabled

To evaluate an LLM pipeline on real historical inputs (e.g. call recordings) without touching production records, clone the workflow and set `disabled: true` on the **leaf write nodes**: CRM patch, score save, cost log. Disabled mid-chain nodes pass data through, so you can read exactly what *would* have been written from the node before the write.
