# Broker & integrations — operate connections, never script them

Two concerns that look similar and must stay apart:

- **Integration SETUP** — *connecting/configuring* a provider. Privileged and
  **human-led**.
- **Broker EXECUTION** — *operating* a connection that already exists. This is the
  agent's lane.

A third lane — **authoring/deploying an integration script** (a Nango action) — is
**human-only**, and neither the broker nor the integration commands touch it.

## SETUP — connecting a provider (human-led)

Done in the dashboard (**Settings → Connections**) or via the CLI connect flow.
The agent does NOT do this on its own initiative.

```bash
driftless integration catalog                 # connectable providers (from Nango)
driftless integration connect <provider>      # start a connect session (returns the Connect UI session)
driftless integration confirm <provider>      # persist the authorized connection + enable its actions
driftless integration list                    # this workspace's integrations
driftless integration rm <id>                 # disconnect / delete an integration
```

API form (privileged connect flow):
`GET …/integrations/nango/catalog` ·
`POST …/integrations/nango/:provider/connect-session` ·
`POST …/integrations/nango/:provider/confirm` ·
`DELETE …/integrations/:id`.

`driftless broker setup` prints exactly these steps. The broker itself never
connects, disconnects, or configures a provider.

## EXECUTION — operating a connection (the agent's lane)

Once a provider is connected, you OPERATE it through the broker. Credentials
resolve **server-side** (they never reach the agent) and **every call is audited**.
Reads and writes both run **inline** — Driftless is a tooling proxy; the
human-in-the-loop is your own agent harness.

```bash
driftless broker connections                              # your connected providers
driftless broker operations <provider> [--refresh]        # the named operations this connection can do
driftless broker invoke <provider> <operation> [--input <json>|@file]   # run one (inline + audited)
driftless broker records <provider> --model <m> [--modified-after <iso>] [--cursor <c>] [--limit <n>]
driftless broker events [provider] [--limit <n>]          # the inbound event feed (a drift signal)
driftless broker criterion <provider> [--add <slug>] [--rm <slug>]      # the team's "how we navigate this"
```

MCP form — one tool, `action` selects the operation:

```text
driftless_broker action:'connections'
driftless_broker action:'operations'        provider:'<provider>'
driftless_broker action:'invoke'            provider:'<provider>' operation:'<op>' input:{...}
driftless_broker action:'records'           provider:'<provider>' model:'<m>'
driftless_broker action:'events'            [provider:'<provider>']
driftless_broker action:'attach-criterion'  provider:'<provider>' topic:'<slug>'
driftless_broker action:'detach-criterion'  provider:'<provider>' topic:'<slug>'
```

(On the CLI the criterion verbs are folded into one subcommand:
`driftless broker criterion <provider> [--add <slug>] [--rm <slug>]`.)

### Operations are pre-enabled, served from materialized metadata

The operations a connection exposes are the **Nango actions enabled for its
integration**, served from materialized metadata (no live round-trip per call).
`--refresh` (CLI) / `refresh:true` re-pulls the live list from Nango. New
operations appear when a human **enables** them in Nango — not by the agent
authoring one.

### Records — the synced-model cursor flow

`broker records <provider> --model <m>` reads a synced model's mirrored records,
**delta-aware**: pass `--modified-after <iso>` for changes since a timestamp, and
page with `--cursor` / `--limit`. The response carries `{ records, nextCursor }`;
broker output is hard-capped so a page never floods context.

### Criterion — read before you invoke

Attach the team's Knowledge to a connection with
`broker criterion <provider> --add <topic-slug>`; read it before acting so the work
follows the team's "how we navigate this provider."

## The no-agent-scripting rule (do NOT cross this line)

A normal agent **operates** connections; it does **not** author or deploy
integration scripts (Nango actions). That lane is human-only.

**If the operation you need is not in `broker operations <provider>`:**

1. Do NOT write a Nango action, a webhook handler, or any script to fill the gap.
2. Do NOT try to connect or reconfigure the provider yourself (that's SETUP).
3. **STOP and report the missing capability** to the human — name the provider and
   the operation you needed, so they can enable the action (or build it) and
   re-run `broker operations <provider> --refresh`.

This boundary is the whole safety model of the broker: governed, audited execution
of capabilities a human has explicitly enabled — never agent-authored code reaching
a live provider.
