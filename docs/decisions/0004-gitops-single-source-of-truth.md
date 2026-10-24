# ADR 0004 · The git repository is the single source of truth

**Status:** accepted · amended by ADR&nbsp;0008 (locked `main`, merge gate, automatic revert on failed checks)

## Context

The failure mode of long-running self-hosted setups is undocumented drift:
changes made by hand on the box, at 1am, months ago, that nobody remembers and
that no file describes. Rebuilding means archaeology.

## Decision

The running platform is a pure function of one git repository. A deploy agent on
the node reconciles the node to `origin/main` on a fixed interval, hard-resetting
the working tree each cycle. Manual changes on the node are transient by design —
the next reconciliation overwrites them.

## Consequences

**Good**

- Rebuilding the node is: provision OS, clone repo, restore data root, let the
  agent converge.
- Every change is a reviewable diff with an author and a message.
- "What is running and why" is answered by reading the repo, always current.
- Rollback is `git revert` + push.

**Bad / accepted**

- Emergency fixes still have to be committed and pushed, not just typed on the
  box — or they vanish on the next tick. This is annoying exactly once per
  incident, and it is the right incentive.
- The deploy agent needs read access to the repo and the ability to run the
  container runtime. That credential is scoped and on-node only; CI never gets
  it.
- Secrets cannot live in the repo, which forces a real secrets-injection story
  (a feature, not a bug).

## Revisit if

Reconciliation latency (the poll interval) becomes a problem — at which point a
push-triggered webhook replaces the poll, same model.
