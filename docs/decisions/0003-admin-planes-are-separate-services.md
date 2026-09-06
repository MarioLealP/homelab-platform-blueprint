# ADR 0003 · Admin surfaces are separate, network-isolated services

**Status:** accepted

## Context

Privileged surfaces — container management, the metrics backend, log viewers,
database consoles — are the highest-value targets on the platform. The tempting
pattern is to expose them at `admin.example.com` behind the same SSO as
everything else.

## Decision

Administrative surfaces are **not** routed by the public edge at all. They live
on a dedicated `admin` network that the edge proxy is not connected to, and are
reached only after the operator connects to the VPN.

## Consequences

**Good**

- The admin UIs are not discoverable or reachable from the internet, regardless
  of SSO. A bug in the SSO layer does not expose them.
- The `edge` and `admin` networks are never joined, so a compromised
  public-facing container cannot reach the container-management API on the same
  network.
- The blast radius of a leaked user session is "the apps that user could reach",
  never "the platform".

**Bad / accepted**

- The operator must connect to the VPN to administer anything. This is a
  deliberate speed bump, and in practice a background always-on tunnel.
- Two access models to keep in your head: public+SSO for apps, VPN for admin.

## Rejected alternatives

- **`admin.example.com` behind SSO** — rejected: still an internet-reachable
  door to the most dangerous surfaces; one forward-auth misconfiguration from
  exposure.
- **IP allow-listing on public admin routes** — rejected: brittle with dynamic
  addresses, and still a public TCP endpoint.
