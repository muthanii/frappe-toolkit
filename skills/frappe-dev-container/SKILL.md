---
name: frappe-dev-container
description: >
  This skill should be used when the user asks to "reproduce this Frappe/ERPNext
  issue in my dev container", "test my fix in the running Frappe dev container", or
  needs a real bench environment to verify OSS contribution work from
  `contribute-to-frappe-oss`. Requires a `frappe/frappe_docker` devcontainer that is
  already running - this skill works inside it, it does not provision or launch one.
metadata:
  version: "0.1.0"
---

# Frappe Dev Container (Issue Repro & Fix Verification)

Work inside an already-running `frappe/frappe_docker` devcontainer to reproduce a
reported bug and verify a fix actually works - real bench behavior, not a guess from
reading the issue. This is scoped to supporting OSS contribution work
(`contribute-to-frappe-oss`): it is not a general-purpose sandbox and should not be
used in place of the `frappe` MCP server for live-site operations.

## Prerequisite: the container must already be running

This skill does not clone `frappe/frappe_docker`, build images, or start the
compose stack - starting a devcontainer is normally a VS Code-managed action the
user drives themselves ("Reopen in Container"), and provisioning one from scratch
here would fight with however they already manage it.

1. Check whether a Frappe bench container is running (e.g. `docker ps`, looking for
   a container from a `frappe_docker`/`devcontainer-example` compose project).
2. If nothing is running, stop and tell the user to start it - point them at
   `frappe/frappe_docker`'s `devcontainer-example` (VS Code "Reopen in Container",
   or bringing its compose stack up directly) - rather than attempting to launch it
   yourself.
3. Once a container is confirmed running, identify it (ask the user for the
   container/service name if more than one candidate is running) and use
   `docker exec` into it for every command below - don't start a second, separate
   stack alongside it.
4. If a bench/site isn't initialized yet inside that running container, do
   first-time setup there (create a bench, create a new site, get/install the app
   the issue lives in, e.g. `erpnext`, at the version/branch relevant to the issue) -
   this is setup *inside* the already-running container, not provisioning the
   container itself.

## Reproducing the issue

- Follow the issue's reported repro steps exactly before changing anything -
  confirm the bug actually reproduces on a clean bench before assuming the fix
  target is correct.
- If it doesn't reproduce as described, say so plainly to the user rather than
  proceeding to "fix" something that isn't actually broken here - the issue may be
  version-specific, data-specific, or already fixed upstream.
- Use `frappe-docs-lookup` to check whether reproduced behavior is actually
  documented/intended before treating it as a bug.

## Verifying a fix

1. Apply the candidate fix inside the container's bench (the mounted app source).
2. Re-run the exact repro steps and confirm the reported behavior is gone.
3. Run the app's existing test suite (or the relevant subset) using the repo's own
   test runner conventions - Frappe apps use `bench run-tests`, not plain
   pytest/unittest invocations.
4. Report concretely what you verified (repro confirmed, fix applied, tests run and
   their result) back into the `contribute-to-frappe-oss` workflow before that
   skill opens a PR. Don't claim a fix is verified if you only read the code and
   didn't actually run it in the container.

## Cleanup

Leave the container running as you found it - this skill didn't start it, so it
shouldn't stop it either. Only tear it down if the user explicitly asks.
