---
name: brein-contact-path
description: Safely activates paid contact research for CRM accounts selected by the user. Use when the user asks for a contact path, email or phone for specific saved accounts, wants a price first, or confirms a quoted credit spend. Do not use for market analysis, bulk lists or accounts that are not already selected in the CRM.
license: MIT
metadata:
  author: Brein
  version: 1.0.0
  mcp-server: driftless
---

# Brein Contact Path

Contact activation is a separate paid action. Analysis never authorizes it.

## Quote

1. Require an exact CRM `collection_id` and 1-50 user-selected `record_ids`. Never infer, expand or replace the selection.
2. Call `driftless_contact_quote`. This is free and does not contact a provider.
3. Show the exact records covered, already-known contacts, maximum credits, current balance and expiry. Ask whether the user approves that specific spend.

## Unlock

Call `driftless_contact_unlock` only after a new, explicit confirmation of the displayed quote. Copy the returned `quote_token`, `idempotency_key`, record ids and approved maximum exactly. Never invent or reuse them for a different selection.

If the user cancels or does not clearly confirm, stop after the quote. If the quote expires, the balance is insufficient or the server refuses a changed price, do not retry the unlock; explain the result and offer a fresh quote.

## Settlement

- Existing contacts are skipped and not charged again.
- Report delivered, skipped and failed records separately.
- On partial failure, retry only incomplete records after a new quote.
- Report where results were stored in the CRM. Do not paste contact coordinates, provider payloads or internal fields into the conversation.
