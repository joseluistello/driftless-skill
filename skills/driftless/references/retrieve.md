# Retrieve — the first move when context is unknown

The whole point of Driftless is *read the team's context before you touch an area.*
This page is the detail behind the routing table in SKILL.md: **which retrieval
call to make**, what each returns, and how to stay cheap (brief) until you need a
full body.

## Pick the call by what you have

| You have… | Call | Why |
|---|---|---|
| The **files** you're about to edit | `driftless context get --files "a.ts,b.ts"` · MCP `driftless_context_get_for_files { files }` | Anchors match → the topics that govern those exact paths, drifted-first |
| Your **local uncommitted diff** | `driftless context get --diff` (add `--mark` to flag matched topics drifted) | Same match-files engine over your working changes — the pre-commit read |
| A **task** but no slug | `driftless context retrieve "<task>" [--explain]` · MCP `driftless_context_retrieve { task, files }` · API `POST /topics/retrieve` | One ranked call: composes search + match-files + list, annotates *why* each hit matched (trust + why_matched) |
| A known **slug** | `driftless context get <slug>` · MCP `driftless_context_get { topic }` | Loads the one topic (full body by default) |
| A **concept / keyword** | `driftless context search <kw>` · MCP `driftless_context_search { query }` | Ranked summaries; read the top few, then `get` one |
| A whole **domain** to browse | `driftless context list --area <name>` · MCP `driftless_context_list { }` | Bounded, relevance-ranked index |

**Don't hand-chain `search → list → get`** when you have a task but no slug —
`driftless_context_retrieve` does it in one call and ranks the results. Reach for
`search`/`list` only when you're deliberately exploring.

## The unified retrieve primitive

`driftless_context_retrieve` (MCP) / `POST /topics/retrieve` (API) is the
START-HERE call. It takes a `task` description, a `files` list, or both, and
returns ranked context:

```text
MCP:  driftless_context_retrieve { task: "add idempotency to the Stripe webhook handler",
                                   files: ["src/billing/webhooks/handler.ts"] }
API:  POST /workspaces/:slug/topics/retrieve
      { "query": "...", "files": ["..."], "view": "brief", "limit": 10 }
```

It returns `{ shown, has_more, next_action, results[] }`, where each result carries
`match_type`, `why_matched`, `confidence`, a `stale` signal, and a `next_action`
hint. Results are **drifted-first** and **bounded** (default 10, max 25). There is
`retrieve` CLI subcommand for the task case — `driftless context retrieve "<task>"`
(query and/or `--files`, `--explain` for why_matched) — plus the file/diff-shaped
reads `context get --files` and `context get --diff`.

## Payload views — stay cheap, opt into heavy

One vocabulary across every read (`summary` / `brief` / `full`):

| View | Carries | Where it's the default |
|---|---|---|
| `summary` | index row: slug, title, trust, badges, anchors — no body | `context list`, `context search` |
| `brief` | the durable *why*: what / decisions / gotchas / invariants — no full `content` | `driftless_context_retrieve`, `driftless_context_get_for_files` |
| `full` | everything, including the heavy `content` body | `context get <slug>` only |

`full` is **never** a default except on a single-slug `get` (you named the one you
want). Everywhere else you opt in (`view: "full"`), and you pair it with a small
`limit`.

**The pattern:** retrieve / get-files in **brief** to see *which* topic governs the
work, then `driftless_context_get` (full) for the **one** body you actually need —
instead of pulling every full body up front.

## Collection retrieve — the operational seam

The same idea on the operational plane. Before working a **record**, read the
collection's **criterion** (the Knowledge the team applies to this work). The
`retrieve` action returns relevant records **and** that criterion in one call:

```text
MCP:  driftless_collection action:'retrieve' id:'<collection-id>'
        [ query | status | entity_id | drifted | updated_after | fields | view | limit | cursor ]
      → { records, nextCursor, criterion, criterion_missing }
API:  GET /workspaces/:slug/collections/:id/retrieve?status=won&limit=25
```

`criterion` is the resolved Knowledge to apply; `criterion_missing` flags any
criterion slug that doesn't resolve (a gap to close). Records paginate by keyset
(`nextCursor`). For just the criterion (no records), use
`driftless_collection action:'context'`. Read the criterion, **then** act:
`driftless_collection_record action:'update' collection_id:'<id>' record_id:'<id>' fields:{...}`.

## After you read

No topic matches the files you're about to edit? That's a gap, not permission to
guess — read the code, then leave one clean note (the six rules in SKILL.md).
Before pushing, run `driftless context get --diff` and refresh any topic your
change drifted.
