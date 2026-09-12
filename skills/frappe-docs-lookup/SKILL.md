---
name: frappe-docs-lookup
description: >
  This skill should be used when the user asks to "look up Frappe docs", "find
  Frappe/ERPNext documentation on X", "how does the Frappe API handle X", "what's
  the right way to do X in the Frappe framework", or needs framework-level reference
  material before writing Frappe app code or calling a Frappe API/method.
metadata:
  version: "0.1.0"
---

# Frappe Documentation Lookup

Use the `frappe` MCP server's documentation tools - `search_frappe_docs` and
`get_frappe_doc` - to ground answers about the Frappe framework in its actual
documentation, rather than relying on memory. Framework APIs and conventions change
across Frappe versions, so a stale recollection can be wrong in ways that break code
on a live site.

## When to use this before other skills

Reach for this skill before:

- Writing or reviewing Frappe app code (Python controllers, client scripts,
  hooks.py, DocType JSON, permissions).
- Calling `frappe_run_method` from the `frappe-site-ops` skill on an unfamiliar
  method - check its documented signature and side effects first.
- Answering "how do I..." questions about Frappe/ERPNext development.
- Advising on Frappe conventions during OSS contribution work (see
  `contribute-to-frappe-oss`).

## Workflow

1. Call `search_frappe_docs` with the user's question or the relevant API/concept
   name to find candidate documentation pages.
2. Call `get_frappe_doc` on the most relevant result(s) to pull the full content.
3. Base the answer on what the docs actually say. If the docs are ambiguous, silent
   on the point, or contradict what the user described happening, say so explicitly
   rather than filling the gap with a guess.
4. Cite the doc page (title and/or path) the answer came from so the user can open
   it themselves for more detail.

## When docs search comes up empty

If `search_frappe_docs` returns nothing useful for a specific, narrow question
(e.g. an internal API not covered in public docs), say so plainly rather than
answering as if you'd found a documented answer. Offer to check the Frappe/ERPNext
source directly (via GitHub, per `contribute-to-frappe-oss`) as a fallback when the
user wants to keep digging.
