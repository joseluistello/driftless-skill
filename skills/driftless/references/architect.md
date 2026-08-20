# Architect mode — grow coverage from YOUR agent, on YOUR inference

The Architect is not a server-side agent: growing the team's recorded context is the
USER's agent's job (you are present when the learning happens, you hold the session
context, and your inference pays for it). This reference is the method. Run it when
the user asks to "grow coverage", "map the repo into topics", or after `context
doctor` / the coverage map shows gaps.

## Start from the coverage map, not a blind crawl

```bash
# The server already knows which source areas have NO topic coverage:
curl -s "$DRIFTLESS_API/workspaces/<slug>/topics/coverage"   # or: the dashboard's Agents view shows the same gaps
```

Work the `uncovered` list. Do NOT crawl the whole repo — the map already did that.

## Method — in order

1. **MAP** the candidate area: list its files, identify the coherent module/boundary.
2. **SKIP what's covered**: `driftless context get --files "<area>/**"` — anything
   already anchored (even under a different slug) is covered. Never duplicate.
3. **READ the key files** — understand WHY it is built the way it is. Never infer a
   claim from file or directory names alone; open the file or drop the claim.
4. **PROPOSE only what passes the admission test** (below), via the normal write path:

```bash
driftless context add <slug> --title "<1-2 words>" \
  --what "<one sentence>" \
  --content @doc.md \
  --area <domain> \
  --pattern "<narrow glob, ~5-40 files>" \
  --status proposed
```

Topics you create are Notes/proposals — an owner/admin merges them into Knowledge.
The server validates your anchors against the checkout and dedupes by slug and
anchor overlap; trust its warnings.

## The admission test (every topic must pass ALL)

- **DURABLE** — a future code change would meaningfully CONTRADICT the claim. A
  paraphrase of the code is not a topic; a transient value is not a topic.
- **The WHY, not the WHAT** — a decision (why it's this way), an invariant (what must
  always hold), or a gotcha (what bites). The code already shows the what.
- **ANCHORED narrow** — real source (skip tests/config/build output/generated files),
  one concept, ~5-40 files. Never a `src/**` catch-all: if you reach for a wide glob,
  the answer is MORE topics, not a wider anchor.
- **GROUNDED** — you can cite a real file you read for every claim. Use `git log`/
  `git blame` for the historical why — you have full history; use it.
- NOT your job: roadmap / launch / process / session notes. That is ephemeral context;
  leave it as a Note or nothing.

## Discipline

- Deterministic slug from the primary anchor dir (`apps/api/src/auth/**` → `api-auth`)
  so repeated passes converge instead of duplicating.
- Bias to UNDER-propose: a missed gap is cheap (the next pass catches it); a spam
  topic erodes the team's trust in the whole vault. 2 real topics beat 4 padded ones.
- Link, don't island: `--rel relates_to:<slug>` / `[[slug]]` to the decisions the new
  topic builds on.
- Check `driftless context search lesson` first — team lessons (approved
  best-practices distilled from past rejections) are rules to follow, not suggestions.
