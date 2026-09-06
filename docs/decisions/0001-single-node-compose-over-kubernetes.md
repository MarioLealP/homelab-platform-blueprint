# ADR 0001 · Single-node Compose over Kubernetes

**Status:** accepted

## Context

The platform runs on one machine and is operated by one person in their spare
time. The workloads are standard containerised web apps and their databases.
Kubernetes (or k3s, Nomad, Swarm) is the reflexive choice for "orchestration",
and would bring rolling deploys, self-healing, and a rich ecosystem.

## Decision

Use plain Docker Compose, one project per service, reconciled from git by a
small agent. No cluster orchestrator.

## Consequences

**Good**

- The mental model is `docker compose up`. Debugging is `docker logs` and
  `docker exec`, not three layers of abstraction.
- Resource overhead is close to zero — all RAM goes to workloads.
- The whole platform definition is readable in an afternoon.
- Upgrades of the orchestrator are not a project.

**Bad / accepted**

- No automatic failover. A hardware fault is downtime until restore. This is
  acceptable given the non-goal of HA and the tested-restore story.
- No rolling deploys — a service blips during its recreate. Acceptable for this
  audience.
- Some upstream projects ship Helm charts and assume Kubernetes; those need a
  Compose translation or are passed over.

## Revisit if

The platform grows past one node, or a workload appears that genuinely needs
horizontal scale or zero-downtime deploys.
