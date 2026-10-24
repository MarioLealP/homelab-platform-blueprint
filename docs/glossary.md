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

**CSR** — Certificate Signing Request. What a workload sends to the CA: its public key and the identity it asks for. The private key never leaves the machine that generated it.

**Deduplication** — a backup tool storing only the blocks that changed since the
last snapshot, so history is cheap to keep.

**Deploy agent** — the small process on the node that fetches the git repository
and brings the running system in line with it. The only thing that deploys.

**Digest pinning** — referring to a container image by its content hash (`image@sha256:…`) instead of a tag. A tag can be moved to point at different content; a digest cannot.

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

**HIDS / NIDS** — Host / Network Intrusion Detection System. A HIDS watches one machine (file integrity, packages, logins); a NIDS watches traffic on the wire. Both detect; neither blocks in this design.

**IdP / Identity Provider** — the single service that holds all accounts and
performs authentication (and MFA) for every other service.

**Ingress** — inbound traffic entering the node from outside, and the rules that
govern it. Only HTTP(S) to the reverse proxy plus a short explicit list of
forwarded ports is allowed.

**Merge gate** — the service that merges pull requests into the locked `main`: routine changes on its own, protected paths and new host power only after the operator approves.

**MFA** — Multi-Factor Authentication. A second proof of identity beyond a
password, enforced by the IdP for every account.

**Minimum release age** — a hold placed on a dependency update so it is only
proposed after it has been public for some days, giving upstream time to catch a
broken or malicious release.

**mTLS** — mutual TLS. Both ends of a connection present certificates, so the server knows which client is calling, not just the other way round.

**Non-goal** — something deliberately excluded from scope, recorded so it is a
decision rather than an omission.

**OIDC** — OpenID Connect. An identity layer on top of OAuth 2.0; the preferred
way for an app to integrate with the IdP.

**Outbound tunnel** — a connector on the node that dials out to an edge provider; public requests come back over that connection, so no inbound web port is open.

**Plane** — a grouping of containers by role and trust level (edge, application,
data, admin, ops), each on its own network.

**Private CA** — a certificate authority run for the platform's own machines. Here: an offline root and two online intermediates in the vault.

**Protected path** — a file or directory whose changes always need the operator to merge, no matter how green CI is: CI, deploy scripts, secrets policy, security-boundary projects.

**Pull-based backup** — the backup repository (or a mediating host) initiates the
transfer and holds the credentials, rather than the node pushing with delete
rights. Limits the blast radius of a node compromise.

**Reconciliation** — the deploy agent's cycle of fetching the repo and bringing
the node's running state in line with it.

**Restore test** — restoring a snapshot into a scratch location and checking the service comes up with intact data. Levels: repository integrity check (automated), manual drill, scheduled automated test (the target).

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

**Unseal** — unlocking a vault's storage after a restart. Here it needs two factors at once (a TPM on the node and a key server on the LAN) and fails closed without them.

**Uptime monitor** — a check that probes public endpoints the way a user reaches them (public DNS, through the edge). Ideally off-node, so it notices when the whole node or uplink is down.

**Vault** — a service that issues and guards credentials. Here it is the private CA and the login point for workload certificates; stack secrets move into it in a later stage.

**WAF** — Web Application Firewall. A filter at the edge that blocks common
web attacks (injection, path traversal, known bad patterns) before they reach an
app.

**Wildcard certificate** — a single TLS certificate valid for every subdomain of
a domain (`*.example`), issued via the ACME DNS challenge.

**WireGuard** — the VPN protocol used for operator access. Connecting places the
operator's device on the internal networks, from where admin UIs are ordinary
internal services.

**Workload identity** — a name for a machine or service, carried in its certificate (a URI such as `urn:…:workload:<name>`), that relying parties check exactly.
