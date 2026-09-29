# Frappe Toolkit

A Cowork/Claude plugin for working with a live Frappe or ERPNext site, looking up
Frappe framework documentation, and contributing to the Frappe open-source project.

## Installing

This repo is its own plugin marketplace (`.claude-plugin/marketplace.json`). Add it
and install the plugin with:

```bash
claude plugin marketplace add muthanii/frappe-toolkit
claude plugin install frappe-toolkit@frappe-toolkit-marketplace
```

Running `claude plugin marketplace add muthanii/frappe-toolkit` again after pulling
new commits re-syncs the marketplace so newly added/updated skills show up.

## Overview

This plugin bundles two MCP servers and five skills that use them:

- Checking connectivity and API credentials against the live Frappe/ERPNext site
- Reading and writing documents on a real Frappe/ERPNext site
- Looking up Frappe framework documentation before writing code or calling APIs
- Reproducing and verifying fixes in a disposable Frappe dev container
- Finding issues in `frappe/frappe`, `frappe/erpnext`, or other `frappe/*` repos,
  fixing them, and preparing pull requests

## Components

| Component | Name | Purpose |
|---|---|---|
| MCP Server | `frappe` | Docker-based Frappe MCP server (`muthanii/frappe_mcp`) for live site data: `frappe_ping`, `frappe_get_doc`, `frappe_search_docs`, `frappe_create_doc`, `frappe_update_doc`, `frappe_delete_doc`, `frappe_run_method`. |
| MCP Server | `frappe-docs` | Node-based Frappe docs MCP server (`muthanii/frappe_docs_mcp`, run from a local clone) exposing `search_frappe_docs` and `get_frappe_doc` against `docs.frappe.io`. No dedicated ping tool - see `frappe-docs-lookup` for how to check the connection. |
| Skill | `frappe-connection-check` | Ping the live site (`frappe_ping`) to confirm the URL and API credentials work, and diagnose connection/credential failures before doing real work. |
| Skill | `frappe-site-ops` | Read/search/create/update/delete Frappe documents and run whitelisted methods, with confirmation for anything destructive. |
| Skill | `frappe-docs-lookup` | Search and read Frappe framework documentation before answering "how does Frappe do X" questions. |
| Skill | `frappe-dev-container` | Work inside an already-running `frappe/frappe_docker` devcontainer (it does not start one) to reproduce a reported issue and verify a fix before opening a PR. Supports `contribute-to-frappe-oss`; not a substitute for the live `frappe` MCP server. |
| Skill | `contribute-to-frappe-oss` | Find a suitable issue in a Frappe org repo, reproduce and fix it (via `frappe-dev-container`), and open a well-formed pull request using GitHub tools. |

## Setup

The bundled `frappe` MCP server runs the `muthanii/frappe_mcp` Docker image and
needs three environment variables set wherever this plugin runs:

| Variable | Description |
|---|---|
| `FRAPPE_URL` | Base URL of your Frappe/ERPNext site (e.g. `https://your-site.example.com`) |
| `FRAPPE_API_KEY` | Frappe API key for a user with the access this plugin should have |
| `FRAPPE_API_SECRET` | Matching API secret for that key |

Generate an API key/secret in Frappe under **User > API Access**. Docker must be
available wherever the plugin's MCP server runs, and it must be able to pull or
already have the `muthanii/frappe_mcp` image.

The `frappe-docs` MCP server has no published Docker image - it's run directly with
Node.js from a local clone:

```bash
git clone https://github.com/muthanii/frappe_docs_mcp.git
cd frappe_docs_mcp
npm install
npm run build
```

Then set one environment variable:

| Variable | Description |
|---|---|
| `FRAPPE_DOCS_MCP_PATH` | Absolute path to that `frappe_docs_mcp` checkout (the one containing `dist/server.js`) |

It also honors two optional variables if you want to override its defaults:
`FRAPPE_DOCS_MAX_BYTES` (response size ceiling, default 100000) and
`FRAPPE_DOCS_CACHE_TTL_MS` (page cache freshness, default 900000). These aren't set
in `.mcp.json` - export them yourself if you want non-default values.

The `frappe-dev-container` skill expects a `frappe/frappe_docker` devcontainer
(from that repo's `devcontainer-example` setup) to already be running before it's
used - it works inside that container via `docker exec`, it doesn't clone, build,
or launch one itself. Start it yourself (e.g. VS Code "Reopen in Container")
before asking for issue reproduction/fix verification. Nothing needs to be added to
this plugin's own config for it.

The `contribute-to-frappe-oss` skill uses whatever GitHub tools/connector are
already available in your session - it does not bundle its own GitHub MCP server.

## Usage

- "Ping Frappe" / "is the Frappe connection working?" → `frappe-connection-check`
- "Look up the Sales Order for customer X in Frappe" → `frappe-site-ops`
- "Create a new Task in ERPNext for..." → `frappe-site-ops`
- "How does Frappe handle child table validation?" → `frappe-docs-lookup`
- "Reproduce this ERPNext bug in a dev container" → `frappe-dev-container`
- "Find a good first issue in frappe/erpnext, fix it, and open a PR" → `contribute-to-frappe-oss` (using `frappe-dev-container` to verify)

## Team use

If you're sharing this plugin with your team, each person needs their own
`FRAPPE_API_KEY` / `FRAPPE_API_SECRET` (scoped to whatever access they should have
on the site) and Docker access to pull `muthanii/frappe_mcp`. `FRAPPE_URL` is
typically shared across the team unless people work against different
sites/environments.

Each person also needs their own local clone+build of `frappe_docs_mcp` with
`FRAPPE_DOCS_MCP_PATH` pointed at it (it's not a pullable image), and Docker plus a
`frappe/frappe_docker` clone for `frappe-dev-container`.
