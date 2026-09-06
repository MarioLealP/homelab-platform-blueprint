# Glossary

**ACME / DNS challenge** — protocol for obtaining TLS certificates automatically.
The DNS challenge proves domain control by writing a DNS record, so no inbound
port is needed and wildcard certificates are possible.

**ADR** — Architecture Decision Record. A short document capturing one decision,
its context, and its consequences. Superseded ADRs are kept and marked.

**C4 model** — a way to draw software architecture at four zoom levels
(Context, Container, Component, Code). This blueprint uses the first two.

**Edge** — the boundary where outside traffic first meets the platform: the
reverse proxy and identity provider.

**Forward-auth** — an authentication pattern where the reverse proxy asks an
external service to validate each request before proxying it. Lets an app with no
auth of its own sit behind SSO.

**GitOps** — operating a system by making the git repository the desired state
and having an agent continuously reconcile reality to it.

**IdP / Identity Provider** — the single service that holds all accounts and
performs authentication (and MFA) for every other service.

**OIDC** — OpenID Connect. An identity layer on top of OAuth 2.0; the preferred
way for an app to integrate with the IdP.

**Plane** — a grouping of containers by role and trust level (edge, application,
data, admin, ops), each on its own network.

**Reconciliation** — the deploy agent's cycle of fetching the repo and bringing
the node's running state in line with it.

**State root** — the single directory tree under which all persistent service
data lives, one subdirectory per service.
