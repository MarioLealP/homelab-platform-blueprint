# ADR 0002 · One edge: reverse proxy with forward-auth, SSO-only

**Status:** accepted

## Context

Many self-hosted apps ship weak auth: no MFA, no lockout, passwords in a local
table, and a login page exposed to the whole internet. Each such page is an
independent attack surface and an independent thing to patch.

## Decision

- All user-facing HTTP goes through one reverse proxy that terminates TLS and
  owns routing.
- Apps that support OIDC use it directly.
- Apps that do not are wrapped in **forward-auth**: the proxy asks the identity
  provider to validate the session before proxying the request.
- Once an app is wired to SSO, its **local login is disabled**. No dual path.

## Consequences

**Good**

- MFA, account lockout, and audit logging are enforced in one place for every
  app, including apps that have no concept of any of that.
- Patching an app's auth is no longer urgent — the proxy already gate-keeps it.
- Adding a service is one route block.

**Bad / accepted**

- The identity provider is now critical path for everything. It gets the same
  backup and monitoring priority as the proxy itself.
- Forward-auth gives coarse allow/deny, not per-object authorisation inside the
  app. Apps that need fine-grained rules must speak OIDC properly.
- A handful of apps behave badly behind a subpath or with header-based identity
  and need per-app quirks.

## Rejected alternatives

- **Leave each app's own login exposed** — rejected: unmonitored, no MFA,
  N attack surfaces.
- **Keep local login as a fallback alongside SSO** — rejected: the fallback
  bypasses MFA and the audit log, so it is the path an attacker uses.
