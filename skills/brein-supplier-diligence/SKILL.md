---
name: brein-supplier-diligence
description: Investigates one supplier or a short supplier shortlist with identity, award history, risk marks and permits from Brein. Use when the user asks who a supplier is, whether it has sold to government, how concentrated its buyers are, or what published risks and expansion signals exist. Do not use for bulk enrichment or contact export.
license: MIT
metadata:
  author: Brein
  version: 1.0.0
  mcp-server: driftless
---

# Brein Supplier Diligence

Build an evidence-backed supplier profile without merging approximate identities.

## Workflow

1. Search with `driftless_market_search_suppliers` using the name, product, territory or RFC supplied by the user.
2. If several candidates could be the same company, show the candidates and ask which one to inspect. Do not guess.
3. Open only the selected candidate with `driftless_market_get_supplier` and its returned `record_ref`.
4. Treat RFC as the strong identity boundary for awards and risk screening. Use `driftless_market_get_supplier_history`, `driftless_market_search_risks` or `driftless_market_screen_risks` only after identity is strong enough.
5. Use `driftless_market_search_permits` when a granted right would materially inform capacity or expansion.

Use currency and amount scope values supported by `driftless_market_capabilities`. Keep named matches labeled as candidates until tied to an RFC.

## Output

Separate:

- verified identity and observed facts;
- evidence-backed patterns, such as buyer concentration;
- published risk marks or permits and what they do not prove;
- unanswered questions and the next useful check.

Never expose contact coordinates, hidden provenance, warehouse identifiers or a bulk supplier export. A missing mark means none was found within declared coverage, not that the supplier is risk-free.
