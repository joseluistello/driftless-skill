---
name: brein-market-map
description: Maps and compares commercial segments with bounded aggregate evidence from Brein. Use when the user asks how many businesses exist, which territory is denser, where a segment is concentrated, or which market to prioritize. Do not use for lists of companies or contact extraction.
license: MIT
metadata:
  author: Brein
  version: 1.0.0
  mcp-server: driftless
---

# Brein Market Map

Turn a market question into a comparable aggregate, not a directory export.

## Workflow

1. Identify the segment, territories and observation type. If a filter or supported value is unclear, call `driftless_market_capabilities` before searching.
2. For one segment and territory, call `driftless_market_count_suppliers`.
3. For multiple segments or territories, call `driftless_market_compare_segments`. Compare like-for-like cells with one `observed_kind`.
4. Explain the strongest differences, the declared coverage and one commercially useful implication.

## Boundaries

- Return counts and comparisons only. Never page supplier search to reconstruct the underlying companies.
- A result describes the observations queried, not an exhaustive census, TAM or proof of current operation.
- Never reveal emails, phone numbers, WhatsApp coordinates or hidden source fields.
- If the request says "all" or demands an exhaustive market, clarify the bounded question instead of widening it silently.

## Output

State the filters used, show the comparison compactly, distinguish evidence from inference, and end with a practical next question or segment to inspect.

If the result is zero, say that no matching observations were found within the stated coverage. Do not say the market does not exist.
