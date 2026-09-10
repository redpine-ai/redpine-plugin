# redpine-plugin

Redpine Connect for Claude Code: licensed, non-public data for AI work in medicine, science, law, and finance, served over MCP.

One plugin, `redpine`, in the marketplace `redpine-connect`. It connects to the hosted Redpine Connect MCP server and ships one skill, `redpine-search`, that teaches Claude how to use it: what the account can reach, how to preview a price before spending, and how to cite what comes back.

## Install

```
/plugin marketplace add redpine-ai/redpine-plugin
/plugin install redpine@redpine-connect
```

Authentication is OAuth against the Redpine Connect server on first use. No token is stored in this repo.

## Layout

```
.claude-plugin/marketplace.json
plugins/redpine/
  .claude-plugin/plugin.json
  .mcp.json                          the hosted server, https://api.redpine.ai/mcp
  skills/redpine-search/
    SKILL.md                         the loop, entitlements, spending, searching, citing
    references/billing.md            selective unlock, free re-fetch, trial, expiry
    references/search.md             filters: which field, the two formats, what to read back
    references/craft.md              querying by data shape
    references/troubleshooting.md    missing tools, no balance, empty, errors
```

## Design rule

The plugin contains no fact the server can answer. Integration names, collection names, prices, and tool lists come from `find-tools` and `list_collections` at runtime, per account. Adding an integration or a collection is a server-side change; this repo does not move.

## Validate

```
claude plugin validate .
claude plugin validate ./plugins/redpine
```

## Other agents

The plugin format is Claude Code specific, but the skill and the server are not. `AGENTS.md` points other agents at the skill.

Codex: add the server to `~/.codex/config.toml`, then open this repo so `AGENTS.md` is picked up.

```toml
[mcp_servers.redpine]
url = "https://api.redpine.ai/mcp"
```

Authentication is the same OAuth flow on first use.
