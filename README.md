# Aranis for Codex

Work with your Aranis organization inside Codex: read vendors and their risk,
control gaps, the asset and process inventory, the risk register, privacy,
business impact analysis and continuity, documents, the Trust Center and BI, and
change them with your confirmation.

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

The server publishes its tools at run time, so Codex always sees the current
list. Grouped by what they cover:

| Area | Read | Change (with confirmation) |
|---|---|---|
| Vendors and assessments | `list_vendors`, `get_vendor_risk_score`, `get_control_gaps`, `get_assessment_progress`, `get_report` | `create_vendor`, `create_assessment` (draft), `send_invitation`, `trigger_reassessment` |
| Action plans | `get_action_plans` | `create_action_plan` (draft), `update_action_plan_item` |
| Inventory | `list_assets`, `get_asset`, `list_business_processes`, `list_products_services`, `list_areas` | `upsert_assets`, `upsert_business_processes`, `upsert_products_services`, `link_assets_to_processes`, `link_processes_to_products`, `set_process_dependencies` |
| Vulnerabilities and scans | `list_vulnerabilities`, `get_cloud_scan` | `import_cloud_assets`, `import_cloud_vulnerabilities`, `update_vulnerabilities`, `start_cloud_scan`, `start_surface_scan` |
| Risk register | `get_risk_register`, `get_risk` | `upsert_risks`, `upsert_risk_treatments` |
| Privacy | `list_processing_activities`, `get_processing_activity`, `list_privacy_risks` | `upsert_processing_activities`, `save_lia`, `save_privacy_report` |
| Business impact and continuity | `list_bia`, `get_bia`, `get_continuity_overview` | `save_bia_impact`, `upsert_recovery_strategies`, `record_continuity_exercise`, `upsert_continuity_plan` (draft) |
| Documents | `list_documents`, `get_document`, `list_document_templates` | `upsert_document` (drafts), `create_document_from_template`, `submit_document_for_review` |
| Trust Center | `get_trust_center` | `upsert_trust_center_content`, `sync_trust_center_subprocessors` |
| BI | `describe_bi_catalog`, `run_bi_query`, `list_bi_panels` | `save_bi_panel`, `update_bi_widget` |
| Reasoning | `ask_assessment_agent` | |

Reading needs the `mcp:read` scope and `ask_assessment_agent` needs `mcp:agent`.
Changing needs `mcp:write`, an OAuth connection (an API key only reads), a role
in the organization that allows that change, and a plan that includes it.

**Every change takes two steps.** The first call writes nothing: it returns what
would change, from what to what, and a confirmation. Only a second call with that
confirmation applies it. The skill below tells Codex to show you the first step
and wait for your yes.

The plugin also ships a skill that tells the model how to read the results —
most importantly that Aranis has two families of score with the **same 0–1 shape
and opposite directions**: risk scores where higher is worse, and maturity and
scanner scores where higher is better. That distinction is the most common way
to misread this data, and it is not guessable from the field names.

## What it does not do

- **It never decides for you.** No change happens without your confirmation of
  the exact preview.
- **Some things are always yours, on the Aranis screen**, never through this
  connection: accepting a risk and signing a risk acceptance letter, approving or
  publishing a document or a plan, publishing the Trust Center, answering a
  Trust Center document request, deleting anything, and anything about people,
  roles, billing or keys. The tools answer with the screen link instead.
- **Trust Center content is public.** FAQ, certifications, roadmap and
  subprocessors have no draft: on a published Trust Center they are visible to
  anyone as soon as you confirm. The preview says whether that is the case.
- **It is bound to one organization**, the one picked at install. It does not
  follow the organization you switch to in the Aranis web app. To use another
  one, authorize again.
- **It does not carry a copy of the tool schemas.** They are discovered from the
  server at run time, so this plugin does not go stale when Aranis adds a tool or
  a parameter.

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
