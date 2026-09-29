---
name: frappe-connection-check
description: >
  This skill should be used when the user asks to "ping Frappe", "check the Frappe
  connection", "is Frappe reachable", "test my Frappe API credentials", "why is the
  Frappe server failing", "check the frappe MCP server", or otherwise wants to
  verify connectivity to the live Frappe/ERPNext site before doing real work.
metadata:
  version: "0.1.0"
---

# Frappe Connection Check

Verify that the plugin can actually reach the live Frappe/ERPNext site and that the
configured API credentials work, before any real read or write operation. Use the
`frappe` MCP server's `frappe_ping` tool as the primary check.

Run this proactively whenever a `frappe_*` call fails unexpectedly, and whenever the
user explicitly asks whether the site is reachable. A failed connection usually
means the problem is configuration (URL, credentials, or Docker), not the operation
the user was trying to perform.

## The check

1. Call `frappe_ping`. A successful response confirms three things at once:
   - the `frappe` MCP server (Docker container) started,
   - `FRAPPE_URL` resolves and the site answers,
   - `FRAPPE_API_KEY` / `FRAPPE_API_SECRET` are accepted.
2. Report the result plainly: either "connected to `<site>`" or the specific failure
   you saw. Do not retry blindly - a second identical call rarely changes the answer.
3. If `frappe_ping` succeeded, the site is reachable and you can proceed with the
   user's actual request (hand off to the `frappe-site-ops` skill for CRUD work).

`frappe_ping` only proves the `frappe` MCP server. It says nothing about the
`frappe-docs` server (docs lookups) - see "Checking frappe-docs" below if the
question was about documentation access.

## When `frappe_ping` fails

Diagnose by the shape of the failure rather than guessing. Read the error text and
map it:

| Symptom | Most likely cause |
|---|---|
| MCP tool/server not found, or a Docker error (`Cannot connect to the Docker daemon`, image pull failure) | Docker isn't running, or can't pull `muthanii/frappe_mcp` |
| Connection refused / DNS failure / timeout / SSL error | `FRAPPE_URL` is wrong, unreachable from this environment, or the site is down |
| HTTP 401 / 403 / "Invalid API key" / "Not permitted" | `FRAPPE_API_KEY` or `FRAPPE_API_SECRET` is wrong, revoked, or the user lacks access |
| HTTP 404 / unexpected page returned | `FRAPPE_URL` points at the wrong path, or the site isn't a Frappe instance |

For each, tell the user the concrete next step (start Docker, fix `FRAPPE_URL`,
regenerate the API key/secret under **User > API Access**, etc.) instead of
continuing as if the request failed for some other reason. Do **not** fall back to
fabricating data or answering from memory when the connection is down - say the site
is unreachable.

See `references/connection-troubleshooting.md` for the full per-symptom checklist.

## Checking frappe-docs

The `frappe-docs` MCP server has no ping tool. To confirm it's reachable, call
`search_frappe_docs` with a broad, cheap query (e.g. `"frappe"`) and treat a normal
result list as confirmation it's up. A startup or connection error instead usually
means `FRAPPE_DOCS_MCP_PATH` is unset/wrong or the checkout hasn't been built
(`npm run build` in `frappe_docs_mcp`).

## Multiple Frappe instances

Each MCP server entry is one instance: the entry named `frappe` carries its own
`FRAPPE_URL` / `FRAPPE_API_KEY` / `FRAPPE_API_SECRET`, and `frappe_ping` only tests
*that* entry. To check more than one site (e.g. staging and production), configure a
separate entry per instance with its own env vars and a distinct name (`frappe`,
`frappe-staging`, `frappe-prod`), then call that entry's `frappe_ping`. Tools are
namespaced by server, so a ping on `frappe-staging` says nothing about `frappe`.

- **Name the instance in every report.** Say which server/site you pinged
  (`frappe-staging (<url>) -> connected`), never a bare "Frappe is up".
- **Pinging several at once:** call each entry's `frappe_ping` and present one status
  row per instance. Diagnose each independently - one instance can return
  "unreachable" while another returns "bad credentials".
- **Writes are the real danger.** With more than one instance configured, never
  assume the default `frappe` server is the intended target. Confirm the instance
  with the user before any create/update/delete or `frappe_run_method`, and state
  which one you acted on.
- **Don't swap a single entry's `FRAPPE_URL` to check several.** That requires
  reloading the MCP config and silently changes which site later operations hit -
  prefer distinct server entries.
- If only one `frappe` entry is configured, there is exactly one instance, and a
  successful ping proves only that site.
