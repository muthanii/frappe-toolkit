---
name: contribute-to-frappe-oss
description: >
  This skill should be used when the user asks to "find a Frappe issue to work on",
  "contribute to Frappe open source", "help me fix a bug in frappe/frappe or
  frappe/erpnext", "find a good first issue in ERPNext", or "open a PR against
  Frappe". Covers finding an issue, understanding it, and preparing a pull request
  using GitHub tools.
metadata:
  version: "0.1.0"
---

# Contributing to Frappe Open Source

Guide the user through contributing to a Frappe organization repository (most
commonly `frappe/frappe` or `frappe/erpnext`, but the same flow applies to any
`frappe/*` app) using the GitHub tools available in this session.

Read `references/contribution-workflow.md` for the full step-by-step workflow,
issue-selection heuristics, and PR checklist before starting a contribution session.

## Quick summary

1. **Pick a repo and confirm scope.** Ask which repo if not stated (`frappe/frappe`
   the framework, `frappe/erpnext` the ERP app, or another `frappe/*` app).
2. **Find a suitable issue.** Search open issues labeled for new contributors (e.g.
   `good first issue`, `help wanted`) via the GitHub tools rather than picking one
   at random - prefer issues with a clear repro or spec over vague ones.
3. **Read before writing code.** Fetch `CONTRIBUTING.md` and any linked coding
   standards for the repo; Frappe's Python style and test conventions differ from
   generic Python projects.
4. **Reproduce and diagnose** the issue locally when possible before proposing a
   fix; don't guess at root cause from the issue title alone.
5. **Open the PR** referencing the issue number, following the repo's PR template,
   and summarizing what changed and why. Never claim a maintainer has pre-approved
   the approach - PR review timelines and outcomes are up to Frappe's maintainers.

Use `frappe-docs-lookup` alongside this skill whenever the fix touches framework
behavior you're not certain about - check the documented behavior before assuming
the issue report is describing a bug rather than intended behavior.
