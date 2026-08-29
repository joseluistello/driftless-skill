---
name: brein-opportunity-radar
description: Finds and verifies public procurement opportunities with buying history and a suggested commercial next step. Use when the user asks what is open, who is buying, where budgets are moving, or which opportunity deserves attention. Do not use for exhaustive tender feeds or generic web news discovery.
license: MIT
metadata:
  author: Brein
  version: 1.0.0
  mcp-server: driftless
---

# Brein Opportunity Radar

Find a small number of defensible opportunities, then verify before recommending action.

## Workflow

1. Translate the request into a product or service, territory, buyer and timing window. If valid filters are unclear, call `driftless_market_capabilities`.
2. Call `driftless_market_search_opportunities` for a bounded shortlist.
3. Open a chosen candidate with `driftless_market_get_opportunity`. Check publication, opening and closing dates before describing it as active.
4. Use `driftless_market_search_awards` or `driftless_market_aggregate_awards` to explain how the buyer or category has purchased. Keep currency and amount scope explicit.
5. Recommend one next commercial action grounded in the evidence, such as qualify, monitor, contact a selected account, or discard.

## Output

For each recommended opportunity, show the signal, buyer, timing, evidence, why it may matter now and the next action. Keep prior awards separate from current opportunities.

Never claim exhaustive coverage. An old publication is not current merely because it appears in search. If dates conflict or the result cannot be verified, label it for confirmation instead of presenting it as open.
