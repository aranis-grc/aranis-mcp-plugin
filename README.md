# Aranis for Codex

Read your organization's third-party risk data from Aranis inside Codex: which
vendors exist, how risky each one is and why, and which controls are missing or
partial.

Sign-in goes through Aranis with OAuth 2.1. **There is no key to paste.** You
choose which organization the access applies to, and you can revoke it at any
time from Aranis.

## Two different things, and it matters which one you want

**Connector** — in ChatGPT on the web, or in Claude: paste
`https://app.aranis.ai/api/mcp`, choose OAuth, done. No plugin, nothing to
install, nothing in this repository. The tools show up and that is the whole
story.

**This plugin** — in the Codex desktop app: installs as a *Codex plugin*. It
carries the same connection, plus a skill that tells the model how to read what
comes back. Same data, different packaging, and only Codex uses it.

If you are not on Codex, you do not need this repository.

## Install in Codex

In the desktop app: **Plugins → Adicionar → Adicionar marketplace de plugins**.

| Field | Value |
|---|---|
| Origem | `aranis-grc/aranis-mcp-plugin` |
| Referência do Git | `main` |
| Caminhos esparsos | leave empty |

Then install `aranis-mcp` from the **Aranis** marketplace that appears.

Codex opens the Aranis sign-in during installation and asks which organization
the access applies to. There is no key to paste.

Sparse paths exist for repositories that keep plugins beside unrelated code.
This one is only the plugin, so the field stays empty.

### Updating

Codex refreshes the marketplace from `main`. The connection URL is the part that
will not change; the skill is. It carries how to read the scores and how to use
the tools, and both follow the product — which is why this lives in a repository
rather than in a file somebody downloaded once.

## What you get

Four read-only tools:

| Tool | Answers |
|---|---|
| `list_vendors` | Which vendors exist, filtered by criticality or assessment status |
| `get_vendor_risk_score` | How risky one vendor is, and the breakdown by risk category |
| `get_control_gaps` | Which controls are missing or partial, with severity |
| `ask_assessment_agent` | Questions that need reasoning over control → risk → impact |

The last one requires the `mcp:agent` scope; the others need `mcp:read`.

The plugin also ships a skill that tells the model how to read the results —
most importantly that Aranis has two families of score with the **same 0–1 shape
and opposite directions**: risk scores where higher is worse, and maturity and
scanner scores where higher is better. That distinction is the most common way
to misread this data, and it is not guessable from the field names.

## What it does not do

- **It is read-only.** Nothing here creates, changes or deletes anything in
  Aranis.
- **It is bound to one organization**, the one picked at install. It does not
  follow the organization you switch to in the Aranis web app. To use another
  one, authorize again.
- **It does not carry a copy of the tool schemas.** They are discovered from the
  server at run time, so this plugin does not go stale when Aranis adds a
  parameter.

## Using Aranis from somewhere else

Any MCP client that speaks OAuth 2.1 connects to
`https://app.aranis.ai/api/mcp` on its own — ChatGPT and Claude among them. This
plugin exists because Codex packages the connection together with the skill
above; the connection itself is not Codex-specific.

For a backend calling the Responses API, the static API key header is the right
credential instead — the caller there is a program, not a person. Aranis →
Settings → API keys has the snippet.

## Revoking

Aranis → Settings. Revoking the authorization kills every token issued under it
immediately.

## Layout

```
.agents/plugins/marketplace.json   the marketplace this repo publishes
plugins/aranis-mcp/
├── .codex-plugin/plugin.json      manifest
├── .mcp.json                      the remote server, https://app.aranis.ai/api/mcp
└── skills/aranis-risk/SKILL.md    how to read what comes back
```

## Support

<https://aranis.ai>
