# 1 · Overview

## Goals

- **One source of truth.** The entire platform — every service, route, and
  network — is described in a single git repository. The running system is a
  function of that repository, not of accumulated manual changes.
- **One way in.** Every human-facing surface is reached through a single
  authenticated edge. There is no service with its own separate login to
  remember, and no admin panel exposed directly to the internet.
- **Recoverable.** Any service can be rebuilt from the repository plus a backup
  snapshot. Restores are tested, not assumed.
- **Boring to operate.** Adding the twentieth service should be the same three
  steps as adding the third.

## Non-goals

- **High availability.** This is a single-node design. It optimises for low
  operational cost and fast recovery, not for zero downtime. Multi-node
  orchestration is explicitly out of scope (see ADR&nbsp;0001).
- **Multi-tenancy.** One operator, a handful of trusted users.
- **Horizontal scale.** If a workload needs a cluster, it does not belong here.

## Platform rules

Every service on the platform follows the same rules. They are what keep a
growing fleet manageable.

1. **Defined in the repository.** A service is a directory with a compose file
   and its configuration. Nothing runs that is not committed.
2. **No host ports unless externally required.** Services talk to each other on
   internal networks. Only the reverse proxy and deliberately port-forwarded
   workloads (game servers, VPN) publish host ports.
3. **Reached through the edge.** User-facing HTTP is routed by the reverse proxy,
   which terminates TLS and applies authentication policy.
4. **Secrets are external.** Configuration in the repo is non-secret. Secrets are
   injected at deploy time from an encrypted store and never committed in clear.
5. **State is on a known path.** Anything that must survive a container rebuild
   lives under one predictable data root, so the backup agent has exactly one
   tree to snapshot.
6. **Least privilege.** Containers run as non-root where the image allows, drop
   capabilities they do not need, and mount the Docker socket only if their
   function genuinely requires it.
7. **Pinned and aged.** Image tags are pinned to a version, not `latest`.
   Dependency updates arrive as reviewable change proposals with a minimum
   release age, not automatically on publish.

## Roles

| Role | Reaches | How |
| --- | --- | --- |
| **User** | Public and SSO-gated apps | Public internet → edge |
| **Operator** | Everything, including admin surfaces | VPN → internal networks |
| **Automation** | Deploy loop, backups, monitoring | Runs on-node, scoped permissions |
