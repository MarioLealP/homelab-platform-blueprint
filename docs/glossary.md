# Glossary

**ACME / DNS challenge** — protocol for obtaining TLS certificates
automatically. The DNS challenge proves domain control by writing a DNS record,
so no inbound port is needed and wildcard certificates are possible.

**ADR** — Architecture Decision Record. A short document capturing one decision,
its context, and its consequences. Superseded ADRs are kept and marked, not
deleted.

**Append-only (backup)** — a backup destination the source node may add to but
cannot modify or delete from. A compromise of the node cannot rewrite or destroy
existing snapshots.

**Bind mount** — mapping a specific host directory into a container, as opposed
to an anonymous volume. Every stateful path is a bind mount under the state root
so that "what persists?" has one visible answer.

**C4 model** — a way to draw software architecture at four zoom levels
(Context, Container, Component, Code). This blueprint uses the first two.

**CI** — Continuous Integration. The automated lint / build / test pipeline that
runs on every change and must pass before a merge to `main`.

**Deduplication** — a backup tool storing only the blocks that changed since the
last snapshot, so history is cheap to keep.

**Deploy agent** — the small process on the node that fetches the git repository
and brings the running system in line with it. The only thing that deploys.

**Drift** — divergence between the running system and the git repository, e.g.
from a manual change on the node. Reconciliation overwrites drift on the next
cycle.

**Edge** — the boundary where outside traffic first meets the platform: the
reverse proxy and identity provider.

**Forward-auth** — an authentication pattern where the reverse proxy asks an
external service to validate each request before proxying it. Lets an app with
no auth of its own sit behind SSO.

**GitOps** — operating a system by making the git repository the desired state
and having an agent continuously reconcile reality to it.

**Health probe** — an endpoint or check the platform calls to confirm a service
is not just running but working. Distinct from an external uptime check.

**IdP / Identity Provider** — the single service that holds all accounts and
performs authentication (and MFA) for every other service.

**Ingress** — inbound traffic entering the node from outside, and the rules that
govern it. Only HTTP(S) to the reverse proxy plus a short explicit list of
forwarded ports is allowed.

**MFA** — Multi-Factor Authentication. A second proof of identity beyond a
password, enforced by the IdP for every account.

**Minimum release age** — a hold placed on a dependency update so it is only
proposed after it has been public for some days, giving upstream time to catch a
broken or malicious release.

**Non-goal** — something deliberately excluded from scope, recorded so it is a
decision rather than an omission.

**OIDC** — OpenID Connect. An identity layer on top of OAuth 2.0; the preferred
way for an app to integrate with the IdP.

**Plane** — a grouping of containers by role and trust level (edge, application,
data, admin, ops), each on its own network.

**Pull-based backup** — the backup repository (or a mediating host) initiates the
transfer and holds the credentials, rather than the node pushing with delete
rights. Limits the blast radius of a node compromise.

**Reconciliation** — the deploy agent's cycle of fetching the repo and bringing
the node's running state in line with it.

**Restore test** — a scheduled drill that pulls the latest backup, stands the
service up against it, and asserts the data is intact. A backup that has never
been restored is only a hypothesis.

**Reverse proxy** — the single front door that terminates TLS, routes each
hostname to the right service, and applies forward-auth. Adding a service means
adding one route block.

**Self-hosted runner** — a CI worker running on infrastructure the operator
controls, rather than a hosted cloud runner.

**Split-horizon DNS** — serving different answers for the same name to on-network
and off-network clients, so local clients can reach services directly. An
optimisation, not a security boundary.

**SSO** — Single Sign-On. One authenticated session at the IdP grants access to
every integrated service.

**Stack** — one `docker compose` project: a group of related containers,
networks, and volumes deployed and versioned together.

**State root** — the single directory tree under which all persistent service
data lives, one subdirectory per service.

**TLS termination** — decrypting inbound HTTPS at the reverse proxy so that
routing and auth act on plain HTTP inside the node's networks.

**Uptime monitor** — an external, off-node check that probes the public
endpoints and the VPN endpoint, so an outage is noticed even when the node's
own internet is down.

**WAF** — Web Application Firewall. A filter at the edge that blocks common
web attacks (injection, path traversal, known bad patterns) before they reach an
app.

**Wildcard certificate** — a single TLS certificate valid for every subdomain of
a domain (`*.example`), issued via the ACME DNS challenge.

**WireGuard** — the VPN protocol used for operator access. Connecting places the
operator's device on the internal networks, from where admin UIs are ordinary
internal services.
