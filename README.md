# Frappe Toolkit

A Cowork/Claude plugin for working with a live Frappe or ERPNext site, looking up
Frappe framework documentation, and contributing to the Frappe open-source project.

## Overview

This plugin bundles a Frappe MCP server and three skills that use it:

- Reading and writing documents on a real Frappe/ERPNext site
- Looking up Frappe framework documentation before writing code or calling APIs
- Finding issues in `frappe/frappe`, `frappe/erpnext`, or other `frappe/*` repos and
  preparing pull requests

## Components

| Component | Name | Purpose |
|---|---|---|
| MCP Server | `frappe` | Docker-based Frappe MCP server (`muthanii/frappe_mcp`) exposing `frappe_ping`, `frappe_get_doc`, `frappe_search_docs`, `frappe_create_doc`, `frappe_update_doc`, `frappe_delete_doc`, `frappe_run_method`, `search_frappe_docs`, and `get_frappe_doc`. |
| Skill | `frappe-site-ops` | Read/search/create/update/delete Frappe documents and run whitelisted methods, with confirmation for anything destructive. |
| Skill | `frappe-docs-lookup` | Search and read Frappe framework documentation before answering "how does Frappe do X" questions. |
| Skill | `contribute-to-frappe-oss` | Find a suitable issue in a Frappe org repo, understand it, and open a well-formed pull request using GitHub tools. |

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

The `contribute-to-frappe-oss` skill uses whatever GitHub tools/connector are
already available in your session - it does not bundle its own GitHub MCP server.

## Usage

- "Look up the Sales Order for customer X in Frappe" → `frappe-site-ops`
- "Create a new Task in ERPNext for..." → `frappe-site-ops`
- "How does Frappe handle child table validation?" → `frappe-docs-lookup`
- "Find a good first issue in frappe/erpnext for me to work on" → `contribute-to-frappe-oss`

## Team use

If you're sharing this plugin with your team, each person needs their own
`FRAPPE_API_KEY` / `FRAPPE_API_SECRET` (scoped to whatever access they should have
on the site) and Docker access to pull `muthanii/frappe_mcp`. `FRAPPE_URL` is
typically shared across the team unless people work against different
sites/environments.
