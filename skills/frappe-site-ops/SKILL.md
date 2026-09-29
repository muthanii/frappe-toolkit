---
name: frappe-site-ops
description: >
  This skill should be used when the user asks to work with a live Frappe or ERPNext
  site through the bundled `frappe` MCP server - phrases like "look this up in Frappe",
  "search Frappe/ERPNext for", "get this Frappe record", "create a Frappe document",
  "update this ERPNext record", "delete this Frappe doc", "run this Frappe method",
  "call a whitelisted method", or "check if Frappe is reachable" / "ping Frappe".
metadata:
  version: "0.1.0"
---

# Frappe Site Operations

Use the `frappe` MCP server's tools to read and write data on a real Frappe/ERPNext
site: `frappe_ping`, `frappe_get_doc`, `frappe_search_docs`, `frappe_create_doc`,
`frappe_update_doc`, `frappe_delete_doc`, and `frappe_run_method`.

## Before doing anything else

If a call to any `frappe_*` tool fails or the site's reachability is in doubt, use
the `frappe-connection-check` skill (`frappe_ping` plus failure diagnosis) to confirm
the connection and credentials are working before troubleshooting further. Report
connection failures plainly (bad URL, bad API key/secret, site down) rather than
retrying blindly.

## Reading data

- Use `frappe_search_docs` when the user describes a record by attributes (customer
  name, status, date range) rather than an exact doctype + name. Ask for the doctype
  if it is not clear from context (e.g. "Sales Order", "Task", "Employee").
- Use `frappe_get_doc` when the doctype and document name (ID) are both known.
- Summarize fetched records for the user in plain language rather than dumping raw
  JSON - call out the fields relevant to what they asked, and offer to show the full
  record on request.
- If a search returns many results, show a short list (name + one or two
  distinguishing fields) and ask the user to narrow down rather than guessing which
  one they mean.

## Writing data - create, update, delete

These are mutating operations against a real system of record. Treat them with care:

1. **Confirm before acting.** State exactly what will be created, changed, or deleted
   (doctype, name, and the fields involved) and get explicit confirmation from the
   user before calling `frappe_create_doc`, `frappe_update_doc`, or
   `frappe_delete_doc` - deletion in particular is destructive and often
   irreversible on a live site.
2. **Validate required fields.** If the user hasn't supplied a field that the doctype
   is likely to require (e.g. `customer` on a Sales Order), ask rather than guessing
   a placeholder value.
3. **Report the result precisely.** After a write, tell the user what actually
   happened (the returned document name/ID, or the exact error) - don't assume
   success just because the call returned without throwing.
4. **Never fabricate a document ID or field value.** If a lookup is needed to get the
   correct reference (e.g. the internal name of a linked customer), fetch it via
   `frappe_search_docs` / `frappe_get_doc` first.

## Running methods

`frappe_run_method` calls a whitelisted server-side method rather than doing plain
CRUD. Before calling it:

- Confirm the method path (e.g. `app.module.doctype.method_name`) and the arguments
  it expects. If unsure, use `frappe-docs-lookup` (a sibling skill in this plugin) to
  check the framework documentation for the method's signature and side effects.
- Treat it as a write operation for confirmation purposes if its name or the user's
  description implies it changes data (submits, cancels, emails, generates a
  document, etc.).

## Multi-site or multi-environment caution

If more than one Frappe/ERPNext instance is configured (e.g. a `frappe` and a
`frappe-staging` server entry - see `frappe-connection-check`), each is a distinct
site with its own URL and credentials. Confirm which instance the user means before
any write operation, use that entry's tools, and state which instance you acted on.
There is no implicit default that is safe for mutating operations.
