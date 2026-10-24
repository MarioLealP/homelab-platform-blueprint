# ADR 0008 · Locked main, a merge gate, and a split CI

**Status:** accepted · amends ADR&nbsp;0004 and the original "a human merges
everything" rule in chapter 5

## Context

Two problems with the original delivery design:

1. **"A human merges everything" did not survive contact with volume.** Weekly
   dependency batches, release bumps and small config changes queued up behind
   one person, so either updates stalled or merges became rubber stamps.
2. **CI was the softest target on the node.** One runner executed pull-request
   code *and* had the host container socket, which made it root for anyone who
   could open a PR, an automated author included.

## Decision

- `main` is locked against direct pushes. Everything arrives as a pull
  request.
- A **merge gate** merges routine changes on its own, fast-forward only,
  against the exact commit that was validated.
- **Protected paths** and **new host power** always wait for the operator.
  Protected paths cover CI, deploy and ops scripts, secrets and their policy,
  and the security-boundary projects. Host power means sockets, capabilities,
  devices, host namespaces, and binds outside the service's own directory.
  The check covers every commit in the PR, not just the net diff.
- CI is split into four roles with separate system users: an untrusted
  **build worker** (rootless, egress-firewalled), a credential-free
  **validator**, the **merge gate**, and a **publisher** with one per-run
  registry credential.
- App releases are pinned by **image digest**. The deploy agent refuses mutable
  tags for release-managed apps.
- The deploy agent **auto-reverts** a commit whose stacks fail their health or
  route checks.

## Consequences

**Good**

- Routine changes ship within about a minute, with no human in the loop, and
  the operator's attention goes to the changes that can actually hurt.
- Running untrusted code in CI no longer reaches anything that matters.
- What was validated is exactly what runs.

**Bad / accepted**

- More machinery: four roles, a policy file, a gate service. These are
  themselves protected paths, so changing them needs the operator.
- The protected-path list must be kept current as projects are added. A new
  security-relevant project that is not on the list would auto-merge.
- Fast-forward-only means a PR that falls behind `main` must be rebased before
  the gate takes it.

## Rejected alternatives

- **Keep human-merges-everything** — rejected: in practice it produced
  rubber-stamp reviews and stale dependencies.
- **Auto-merge everything that passes CI** — rejected: a green CI run says
  nothing about whether a change widens host access or edits the deploy
  scripts.
