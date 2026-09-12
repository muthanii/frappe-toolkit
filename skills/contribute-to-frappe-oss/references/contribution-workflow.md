# Frappe OSS Contribution Workflow

Detailed reference for the `contribute-to-frappe-oss` skill.

## 1. Confirm the target repository

Common targets:

- `frappe/frappe` - the core framework
- `frappe/erpnext` - the ERP application built on the framework
- Any other `frappe/*` app (e.g. `frappe/hrms`, `frappe/lms`, `frappe/helpdesk`)

If the user hasn't named one, ask. Don't assume `frappe/frappe` by default - most
day-to-day bugs users hit live in `frappe/erpnext` or another app.

## 2. Finding an issue

Search issues using the GitHub tools available in this session. Prioritize:

- Labels like `good first issue`, `help wanted`, `bug` with a clear reproduction.
- Issues with recent maintainer activity (not stale/abandoned threads) unless the
  user specifically wants to revive an old one.
- Issues that match the user's stated interest (a specific module, doctype, or bug
  type) over a generic "any issue" ask.

Avoid issues that are actually feature requests still under design discussion
unless the user wants that kind of work - those are higher-risk for a first
contribution since scope can shift underneath a PR.

## 3. Before writing any code

- Fetch and read `CONTRIBUTING.md` (and `CODE_OF_CONDUCT.md` if present) from the
  target repo.
- Check for a repo-specific style guide (Frappe has documented Python/JS
  conventions - naming, docstring style, whitelisting rules for API methods).
- Check whether the issue already has a linked PR or an assignee - avoid duplicate
  work.
- Use `frappe-docs-lookup` to verify framework behavior mentioned in the issue
  before assuming it's a bug rather than documented behavior.

## 4. Reproducing, fixing, and verifying

Use the `frappe-dev-container` skill for this step - it works inside an
already-running `frappe/frappe_docker` devcontainer to reproduce and verify
against, rather than guessing from the issue text or touching the live site behind
the `frappe` MCP server. If no such container is running yet, ask the user to start
one first (that skill does not launch it).

- Reproduce the reported behavior in the container before proposing a fix. If it
  doesn't reproduce, say so rather than assuming the fix target is correct.
- Trace the bug to its root cause rather than patching the symptom - Frappe issues
  often surface in a UI layer but originate in a shared framework method used by
  many doctypes.
- Apply the fix in the container, re-run the repro steps, and run the app's tests
  via its own test runner conventions (`bench run-tests`, not plain
  pytest/unittest) before considering the fix verified.

## 5. Opening the pull request

- Fork the repo (or branch directly if the user has write access) using the GitHub
  tools.
- Use a descriptive branch name referencing the issue (e.g. `fix-1234-sales-order-validation`).
- Follow the repo's PR template if one exists.
- Reference the issue number in the PR description (e.g. `Fixes #1234`).
- Summarize: what was broken, why, what the fix does, and how it was tested.
- Be upfront with the user about what's uncertain - don't claim CI will pass or that
  maintainers will approve; those are outcomes to wait for, not guarantees to make.

## 6. After opening the PR

- Watch for review comments and relay them to the user rather than silently
  reworking the PR based on assumptions about what a reviewer wants.
- If CI fails, diagnose the actual failure from the logs rather than guessing at a
  fix.
