# 7. Evaluating Third-Party APIs Before Integration

An agent is only as reliable as the tools it calls. Before giving an LLM a new tool backed by a third-party API, evaluate the API **outside the agent**, with a throwaway probe workflow.

Case study: a search-and-price API for a bookable product, evaluated for an agent that quotes prices to customers. **Verdict: not ready.**

---

## Silent failures that return HTTP 200

| Input | Response | Why it's dangerous |
|---|---|---|
| Date in a regional format (`01/10/2026`), an invalid date, even the word `tomorrow` | `success: true`, real results **for tomorrow** | An LLM writing a locale-formatted date gets a perfect quote for the wrong day |
| Invalid location code | `success: true`, 0 results | Looks exactly like "nothing available" |
| Bad parameter on a calendar endpoint | `{"message":"Success","status":true,"code":500, "data":{"Error":…}}` | An error disguised as success |
| Round-trip search, every route tried | `success: true`, 0 results | The most common customer request silently returns nothing |

**Rule:** error detection reads `body.status` **and** nested error objects, **never the HTTP code alone**. Validate inputs (dates, codes) yourself before calling.

---

## Prices that don't reproduce

Same item, same moment, requests differing only in **sort order**:
- the **base price** was identical every time
- the **tax and fee component** swung by ~44%
- the **total the customer pays** moved by ~9%

It affected every result on the page uniformly, so it was a price-assembly defect, not market volatility. Fetching every page stabilised the **set of results** but not the **price**.

If the number the agent would quote isn't reproducible, the agent can't quote it.

---

## Can the API express what you sell?

The sales team always quotes a specific bundle (included extras and conditions). The API's results often didn't include that bundle, or showed it inconsistently. **An accurate API for the wrong product is still the wrong tool.** Check that the API can represent your actual offer before checking anything else.

---

## The evaluation checklist

1. **Map the endpoints and scopes.** Does the existing credential already cover them?
2. **Reproducibility:** same request ×5, then vary only non-semantic parameters (sort, page).
3. **Drift:** same request a few minutes apart.
4. **Malformed inputs:** every format an LLM might produce, plus invalid codes and edge dates.
5. **Error shapes:** what does failure look like in the body?
6. **Coverage:** the routes and products customers actually ask for, not the demo ones.
7. **Product fit:** can it represent what you sell?
8. **Verdict + what would change it**, written down so the evaluation can be re-run when the vendor updates.

Do it in a **separate, archived probe workflow**, never inside the live agent.
