# Connection Troubleshooting Checklist

Detailed diagnosis for when `frappe_ping` (or any `frappe_*` tool) fails. Work top
to bottom: the first failing check is your answer.

## 0. Confirm what you're testing

- Which MCP server? `frappe_ping` tests one server entry only. Docs access is a
  separate server (`frappe-docs`) with no ping tool.
- Which instance? If more than one `frappe*` entry is configured (e.g. `frappe` and
  `frappe-staging`), each is a distinct site with its own URL and credentials. Note
  which one you're pinging, and don't assume the default entry is the site the user
  means.
- Which site? The server targets whatever `FRAPPE_URL` that entry is set to. Note it
  down so the user can confirm it's the environment they expect.

## 1. Is Docker available and can it run the image?

The `frappe` MCP server is launched as `docker run ... muthanii/frappe_mcp`.

- `docker version` (or `docker info`) - fails with "Cannot connect to the Docker
  daemon" if Docker isn't running or the user lacks access.
- `docker image ls muthanii/frappe_mcp` - if absent, the plugin needs network access
  to pull it; a pull failure points at registry/network issues.
- Fix: start Docker (or grant the user Docker access) and ensure the image is
  pullable. This is the fix for "server failed to start" errors.

## 2. Is `FRAPPE_URL` correct and reachable?

- The URL must be the site **base** (e.g. `https://your-site.example.com`), no
  trailing `/app` or `/api`.
- From the environment running Docker, test reachability directly:
  `curl -sS -o /dev/null -w '%{http_code}\n' <FRAPPE_URL>/api/method/ping`
  - Connection refused / DNS failure / timeout -> wrong host, wrong network, or site
    down. Note that `localhost` inside a container is the *container*, not the host -
    a host site needs `host.docker.internal` or the host IP.
  - SSL/TLS errors -> bad certificate, or a self-signed cert the client won't trust.
  - 404 -> URL likely points at the wrong path or isn't a Frappe site.

## 3. Are the credentials valid?

- `FRAPPE_API_KEY` and `FRAPPE_API_SECRET` must be the matching pair for one user.
- Generate/rotate in Frappe under **User > API Access** (the user row for the API
  account). Regenerating the secret invalidates the old one.
- 401/403 or "Invalid API key" / "Not permitted" means the key/secret is wrong,
  revoked, or the user lacks permission for that call. A ping that authenticates but
  a specific operation that 403s is a *permissions* problem, not a connection one.
- Confirm the API user actually has the roles needed for the operation; ping may
  succeed with a read-only user while a write later fails.

## 4. Still failing?

- Re-run `frappe_ping` once after any config change - don't loop.
- If the error text doesn't match the patterns above, quote it verbatim to the user
  and report that the cause is unclear rather than guessing.
- Never substitute fabricated or remembered data for a live lookup when the
  connection is down. State plainly that the site is unreachable.
