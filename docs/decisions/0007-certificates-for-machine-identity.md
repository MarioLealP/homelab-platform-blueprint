# ADR 0007 · Machines authenticate with short-lived certificates, not shared tokens

**Status:** accepted

## Context

Service-to-service calls, and calls from desktop agents to server APIs, were
authenticated with long-lived shared bearer tokens. Each token sat in two or
more config files, never expired, and proved only "someone who has this
string", not *which* machine was calling. Rotating one meant editing every copy
by hand, so it never happened.

## Decision

- Each workload gets a **named identity** in a URI in its certificate, issued
  by a private CA in a vault (offline root, separate intermediates for vault
  login and for service mTLS).
- Service certificates live for **about two hours** and renew automatically.
  Login certificates live for about 90 days.
- Workloads generate their own keys and submit CSRs. Private keys never move.
- Relying parties check the **exact expected identity**, not just the issuer.
- Revocation is short lifetimes plus quarantine plus a serial deny list. The
  proxy does not do CRL/OCSP, so the design doesn't pretend it does.
- Migration is staged. The vault handles identity first. Stack secrets move
  only after its audit, recovery and rotation are built and tested.

## Consequences

**Good**

- A stolen service certificate expires within hours. A stolen login
  certificate can be quarantined at once.
- Every request carries a verifiable "who", which ends up in the app's logs.
- Shared tokens are deleted, not kept as a fallback, so there is no second way
  in.

**Bad / accepted**

- The vault is now on the critical path for renewals. If it is sealed for
  longer than a certificate's lifetime, the links between machines stop. Hence
  the automatic two-factor unseal and its monitoring.
- More moving parts per workload: a renewer, a roster entry, an enrolment
  step.
- Components that cannot do mTLS need a documented, narrower exception.

## Rejected alternatives

- **Keep shared tokens, rotate them** — rejected: rotation without automation
  doesn't happen, and a token still doesn't say who is calling.
- **Put the vault in charge of all secrets on day one** — rejected: moving every
  credential into an unproven system at once is how a platform gets locked out
  of itself.
