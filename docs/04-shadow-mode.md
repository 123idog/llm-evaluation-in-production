# 4. Shadow Mode

**The bot computes what it would have said. The human answers as usual. A separate job compares the two.** Nothing is sent and the live bot isn't touched.

---

## When shadow mode beats sign-off

The original plan for a pricing feature was for agents to **approve a rate card** before the bot used it. They didn't engage. In practice, people asked to sign off on automating their own work rarely do.

Shadow mode removes the dependency:
- **Zero customer risk:** the shadow sends nothing.
- **Zero cooperation needed:** it reads the replies agents already send.
- **Ground truth for free:** every agent quote is a labelled example.

---

## Shape

```
Every 30 min
  → read recent HUMAN replies (from the log of agent messages)
  → collapse to one item
  → read recent shadow rows (for dedup)
  → compute what the bot WOULD have answered, for each human reply in scope
  → write: human answer, bot answer, match?, reason
```

- **Read from the human-reply log, never from the bot.** It's a separate workflow with no send node. Check the deployed JSON to confirm.
- **Dedup by message key** (`agent_msg_id + line index`). The read window overlaps the schedule, so without dedup every quote is counted 2–6 times.

---

## Tiered confidence

Don't shadow everything. Only quote where history says you can:

| Tier | Rule | Bot behaviour |
|---|---|---|
| **1** | ≥80% agreement over ≥20 historical quotes | Shadow-quote (later: answer) |
| **3** | Anything else | Hand to a human |

Explicit **gates** send cases to tier 3 even on a tier-1 route: per-person pricing wording, extras (child seat, extra stops, premium vehicle, hourly), unrecognised places, multi-stop trips.

In one case, tier 1 covered **about half of demand** with only a handful of rules. That's a good trade: automate the frequent, simple half well, and leave the long tail to humans.

---

## Bugs only shadow-style evaluation catches

**A missing spelling variant gave a confident wrong answer.** A place name written without one letter wasn't in the lexicon, so a long transfer collapsed to "airport ↔ hotel" and would have been quoted at **half the real price, with full tier-1 confidence.** The fix was the lexicon entry plus a gate for unrecognised destination types.

The general lesson: **the dangerous errors are the confident ones.** Shadow logs make them visible, because every bot answer sits next to a human answer.

---

## Reading shadow results

- Track **exact match** and **within X%** separately.
- Read the mismatches by hand. Many are humans being inconsistent, not the bot being wrong. Those are methodology questions for the team.
- The number you're after is the **out-of-sample** agreement rate on live traffic. That's the figure you can defend when asking to switch the bot on.

---

## Shadow for new bots, too

The same pattern evaluates a whole new bot version: run it in shadow on live inbound traffic, compare its replies and its CRM decisions with what the current bot or the humans did, and promote it only when the comparison holds on **real traffic**, not only on a replay set.
