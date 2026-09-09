---
name: aranis-risk
description: Read third-party risk data from the user's Aranis organization — which vendors exist, how risky each one is and why, and which controls are missing. Trigger on Aranis, vendors or suppliers, third-party risk, TPRM, control gaps, assessment findings, or supplier risk scores.
---

# Aranis third-party risk

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

1. `list_vendors` first when the user names a vendor in prose. Names are
   ambiguous and the other tools take an id.
2. `get_vendor_risk_score` for how risky a vendor is, including the breakdown by
   risk category — that breakdown is what answers "why", and the answer is
   usually more useful than the number.
3. `get_control_gaps` for what is missing or partial. Severity comes from CVSS,
   so higher is worse.
4. `ask_assessment_agent` only when the question needs reasoning over the causal
   chain — control → risk → impact — rather than reading a field. It costs more
   and it requires the `agent` scope; if the account does not have it, say which
   capability is missing instead of retrying.

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

## If a write tool ever appears

This connection is read-only today. If the server ever exposes a tool that
creates, updates, submits or deletes, ask the person before calling it, and say
exactly what it will change. Read-only calls need no confirmation.

## When authorization fails

If a call comes back unauthorized, the usual causes are that the consent was
revoked, or that the person no longer has access to the organization this
connection is bound to. Ask them to re-authorize the plugin rather than
retrying.
