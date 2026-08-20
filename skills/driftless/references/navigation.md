# Navigation — moving across planes without listing everything

Driftless has four planes: **Knowledge** (topics), **Projects** (the loop),
**Collections** (records), **Broker** (connections). This page is how you move
*between* them on a real task without ever enumerating a plane. The detail behind
the rule lives in `docs/performance/navigation.md`; this is the agent's copy.

## The one rule

**Retrieve, don't enumerate.** At scale a `list-all` of any plane is wrong (a
50k-card project, a 50k-record collection, dozens of connections). Every step is
bounded and tells you the next bounded step in its `next_action`.

## The bounded path (stop at the shallowest step that answers you)

| Step | Plane | Call | You get |
|---|---|---|---|
| 1 | Knowledge | `driftless_context_retrieve { task, files? }` | the governing topics/notes |
| 2 | Projects | `driftless_project_card { action:"next"\|"list", project_id }` | the actionable card + its context bundle |
| 3 | Collections | `driftless_collection { action:"retrieve", id }` | bounded records page + the criterion to read FIRST |
| 4 | Broker | `driftless_broker { action:"operations", provider }` | the bounded operation list for that connection |
| 5 | Act | `driftless_broker action:"invoke"` · `driftless_collection_record action:"update"` | the write — only AFTER the drill-down |

Enter at the plane that matches what you already have (a slug → `context_get`; a
known card → step 2; a known connection → step 4). Most tasks never reach step 5.

## What this forbids

- **No client-side scan to find one thing.** Need a specific card / record /
  operation? Use a *filter* call (query + keyset cursor), never a full `list` you
  then grep in your head.
- **No drilling past what you need.** Read until you can act, then act once.
- **No inventing a traversal.** Follow the `next_action` rail. If a plane can't
  filter server-side, STOP and report it — don't paper over it with list-all.
- **No scripting across a seam.** Crossing planes is the `next_action` chain, not a
  Nango action you author (the no-scripting rule still holds — see `broker.md`).
