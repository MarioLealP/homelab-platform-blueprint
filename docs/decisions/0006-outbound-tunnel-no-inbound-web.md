# ADR 0006 · Public web through an outbound tunnel, no inbound web ports

**Status:** accepted

## Context

The original design opened 80/443 on the node and pointed DNS at it. That
puts a listening web port on a residential IP, where it is scanned within
minutes, and makes the node's own uplink the thing that absorbs any flood. Most
services here are for the operator alone and never needed to be public at all.

## Decision

- No inbound HTTP(S) port is forwarded to the node.
- A connector on the node opens an **outbound** tunnel to the CDN/edge
  provider. A short, explicit list of hostnames is served over it.
- Everything not on that list is reachable only from the LAN or the VPN.
- The only port forwards are for non-HTTP needs: the VPN endpoint and game
  servers.
- The reverse proxy behind the tunnel stays the single place routing and
  authentication policy are defined.

## Consequences

**Good**

- Nothing on the node answers an internet port scan on 80/443.
- Geo rules, rate limits and flood absorption happen at the provider, before
  traffic reaches the home uplink.
- Making a service public is a deliberate, reviewable one-line change to the
  tunnel config (a protected path, so the operator approves it).

**Bad / accepted**

- The provider terminates TLS for tunnel hostnames and sees that traffic in
  plaintext. Accepted for the few public hostnames, and the reason most
  services are not public.
- The edge-WAF and IP-reputation tooling on the node now sees the tunnel
  connector as the client unless the proxy is told to trust it and read the
  real client IP from the forwarded header. If the connector's network is
  recreated, that trust setting must follow, or the WAF goes blind.
- An outage at the provider takes the public hostnames down. LAN and VPN access
  are unaffected.

## Rejected alternatives

- **Open 80/443 with a WAF in front** — rejected: still a scanned port on a
  home IP, and the uplink still takes the flood.
- **VPN-only for everything** — rejected for the few things that must be
  reachable by people without a VPN client: sign-in, the git forge, small
  public apps.

## Revisit if

Plaintext at the provider becomes unacceptable for a public hostname. The
fallback is direct 443 for that hostname only, with the same proxy and WAF.
