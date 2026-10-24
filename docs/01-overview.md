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
2. **No host ports unless externally required.** Each app sits on its own
   network shared only with the reverse proxy. Only deliberately forwarded
   non-HTTP workloads (game servers, VPN) publish ports to the outside.
   Public web arrives through an outbound tunnel, not an open port.
3. **Reached through the edge.** User-facing HTTP is routed by the reverse proxy,
   which terminates TLS and applies authentication policy.
4. **Secrets are never in clear.** Secrets are committed only in encrypted form
   and rendered at deploy time. Human passwords live in a separate offline
   password manager.
5. **State is on a known path.** Anything that must survive a container rebuild
   lives under one predictable data root, so the backup agent has exactly one
   tree to snapshot.
6. **Least privilege.** Containers run as non-root where the image allows, drop
   capabilities they do not need, and mount the Docker socket only if their
   function genuinely requires it.
7. **Pinned and aged.** Image tags are pinned to a version, not `latest`, and
   app releases are pinned by digest. Dependency updates arrive as pull
   requests after a minimum release age, never automatically on publish.
8. **Machines have names, not passwords.** Service-to-service and
   agent-to-server calls authenticate with short-lived certificates that carry
   a specific workload identity, not shared bearer tokens.
9. **Changes that widen power wait for a human.** Routine changes merge
   themselves. Anything touching the security boundary, the deploy machinery
   or a service's access to the host waits for the operator.

## Roles

| Role | Reaches | How |
| --- | --- | --- |
| **User** | Public and SSO-gated apps | Public internet → edge |
| **Operator** | Everything, including admin surfaces | VPN → internal networks |
| **Automation** | Deploy loop, CI roles, backups, monitoring | Separate system users, scoped permissions, audited |
| **Workloads** | The specific services they are allowed to call | Client certificate with their own identity |
