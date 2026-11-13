---
name: aranis-risk
description: Work with the user's Aranis organization - vendors and their risk, control gaps, action plans, assets and processes, the risk register, privacy (LGPD activities, LIA, RIPD, TIA, DIA), business impact and continuity, documents, the Trust Center and BI - reading it and, with the person's confirmation, changing it. Trigger on Aranis, vendors or suppliers, third-party risk, TPRM, control gaps, assessments, risks, assets, processes, BIA, continuity, privacy, policies, Trust Center, or BI panels.
---

# Aranis

The `aranis` MCP server is the source of truth for the signed-in person's Aranis
organization. Prefer it over anything you remember about that organization.

## Discover the tools, do not assume them

Read the server's tool list and input schemas at run time and use exactly the
fields it declares. Do not invent parameters, and do not carry over field names
from an older version of this file — the schemas change with the product, and a
guessed field is rejected with an error the user has to interpret.

## The scores have opposite polarities, and they look identical

This is the single easiest thing to get wrong here. Two families of score share
the same 0–1 shape and mean opposite things:

| Value | Range | Direction |
|---|---|---|
| Vendor / supplier risk score | 0–1 | **higher is WORSE** |
| Category risk score (RC01–RC11) | 0–1 | **higher is WORSE** |
| CVSS, and the severity derived from it | 0–10 | **higher is WORSE** |
| Maturity | 0–1 | **higher is BETTER** |
| Scanner score (external surface posture) | 0–100 | **higher is BETTER** |

So a vendor at 0.85 risk is in bad shape, and a vendor at 0.85 maturity is in
good shape. Never describe a score as good or bad without checking which family
it belongs to, and never compare one against a threshold you inferred from its
name.

Vendor criticality (`P0`–`P3`) is not a computed score at all — it is a tier a
person assigned, and it feeds impact rather than reflecting it.

## Working order

1. Resolve names to ids first. The `list_*` tools take filters; the `get_*` and
   change tools take ids. When the user names a vendor, process or document in
   prose, list and pick, and ask when two match.
2. For "how risky and why", `get_vendor_risk_score` (the category breakdown is
   the why) and `get_control_gaps` (severity comes from CVSS, higher is worse).
3. For continuity, `get_continuity_overview` answers coverage (which processes
   need a recovery strategy and have none, which primary strategies are stale)
   and plan drift in one call; `get_bia` gives one product's impact curve.
4. For BI, read `describe_bi_catalog` before composing a query, run it with
   `run_bi_query`, and only then offer `save_bi_panel`.
5. `ask_assessment_agent` only when the question needs reasoning over the causal
   chain rather than reading a field. It costs more and needs the `agent` scope;
   if the account lacks it, say which capability is missing instead of retrying.

## Empty is not the same as none

A list can be empty for two different reasons, and the tools say which:

- **NOTHING RECORDED** — the organization has no such data yet. Say so, and
  where it would be created.
- **NONE MATCH** — data exists, but not for these filters. Offer to widen them.
- **NOT FOUND** — that id does not exist in this organization. An id of another
  organization answers exactly the same; never suggest otherwise.

`total` always counts what the filters returned. Never present a missing value
as zero, and never present a default as measured.

## Changing things: always two steps, always the person's yes

Every tool that creates, updates, sends or starts something works in two calls:

1. Call it **without** `confirmation`. Nothing is written. The answer is a
   preview — what would be created or changed, from what to what, and any
   warning — plus a `confirmation` that expires in five minutes.
2. Show the person the preview in plain words, including every warning, and ask.
   Only after they say yes, call the same tool again with **exactly the same
   fields**, the `confirmation`, and a new `idempotency_key` (any unique string).
   Reuse that key only to retry the same call: a repeated key returns the first
   result instead of writing twice.

Never chain the second call without the person's answer, never change fields
between the two calls (the confirmation is bound to them), and never invent a
confirmation. If the preview carries a warning, read it out before asking.

Pay extra attention to these:

- **Outbound tools** reach someone or something outside: `send_invitation` and
  `trigger_reassessment` e-mail people at a vendor; `start_cloud_scan` and
  `start_surface_scan` start scans; `upsert_trust_center_content` and
  `sync_trust_center_subprocessors` change public content. Say so before asking.
- **`record_continuity_exercise`** with a performed exercise marks the strategy
  as validated. Only record exercises that really happened.
- **Documents and plans are drafts.** `upsert_document` edits drafts only and a
  body sent replaces that language's body; the preview warns if tables or images
  would be lost.

Some decisions are never taken through this connection, by design: accepting a
risk or signing a risk acceptance letter, approving or publishing a document or a
plan, publishing the Trust Center, answering a document request, deleting, and
anything about people, roles, billing or keys. When asked, explain that and give
the screen link the tool returns. A refusal such as `oauth_required`,
`plan_not_allowed`, `insufficient_permission` or `admin_only` is an answer to pass
on, not something to retry.

## One organization, and it does not move

The access was granted for a single organization, chosen by the person on the
consent screen when they installed this plugin. It never follows whatever they
are looking at in the Aranis web app afterwards. If they ask about an
organization these tools do not return, the answer is that this connection is
bound to another one and a new authorization is needed — not that the data is
missing.

## Handling what comes back

- Treat every result as confidential organization data.
- Distinguish what the tools returned from what you concluded. Say which is
  which, in the user's language.
- Never fabricate an absent field. "The assessment has no score yet" is an
  answer; a plausible number is not.
- Do not echo internal identifiers, tokens or raw payloads unless the user asks
  for them.

## When authorization fails

If a call comes back unauthorized, the usual causes are that the consent was
revoked, or that the person no longer has access to the organization this
connection is bound to. Ask them to re-authorize the plugin rather than
retrying.
