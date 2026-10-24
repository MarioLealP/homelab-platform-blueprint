# Homelab Platform Blueprint

A reference architecture for running a small fleet of self-hosted services on a
single node — reverse proxy, single sign-on, a dozen or two containerised apps,
their databases, backups and monitoring — without it turning into a pile of
unmanaged containers.

*Updated 2026-10: outbound-tunnel ingress, per-app networks, certificate-based
workload identity, a merge gate with a split CI, and a detection layer. See
ADRs 0006–0008.*

This repository is **documentation only**. It describes a design and the
reasoning behind it; it is not a deployment of any particular system and contains
no hostnames, addresses, credentials or environment-specific configuration. Tool
names (Caddy, Authentik, Restic, …) appear as concrete examples of a category,
not as requirements — the shape of the architecture is what matters.

## Why this exists

Self-hosting tends to grow one `docker run` at a time. Six months in there is no
single source of truth, no consistent way in, secrets live in shell history, and
nobody is sure the backups restore. This blueprint is the opposite: one
git-managed definition of the whole platform, one authenticated entry point, one
backup story, and a short list of rules that every new service follows.

## The design in one diagram

```mermaid
flowchart TB
    subgraph internet [Internet]
        user([User])
        admin([Operator])
        cdn[Edge provider]
    end

    subgraph edge [Edge]
        tunnel[Tunnel connector<br/>outbound only]
        proxy[Reverse proxy<br/>TLS · routing · forward-auth · mTLS]
        idp[Identity provider<br/>OIDC / forward-auth]
    end

    subgraph apps [Application plane]
        pub[Public apps]
        priv[SSO-gated apps]
    end

    subgraph adminplane [Admin plane · not internet-facing]
        mgmt[Management surfaces<br/>container mgmt · dashboards]
    end

    subgraph data [Data plane]
        db[(Databases)]
        obj[(Object / file storage)]
    end

    subgraph ops [Operations]
        backup[Backup agent<br/>pull-based, offsite]
        mon[Monitoring &amp; alerting]
        vault[Vault / private CA<br/>workload certificates]
    end

    user --> cdn
    tunnel -->|dials out| cdn
    cdn -.-> tunnel
    tunnel --> proxy
    admin -->|VPN only| mgmt
    proxy --> pub
    proxy --> idp
    proxy -->|auth check| priv
    idp -.->|verifies| proxy
    pub --> db
    priv --> db
    priv --> obj
    backup -.->|reads snapshots| db
    backup -.->|reads snapshots| obj
    backup --> offsite[(Offsite repository)]
    mon -.->|scrapes| apps
    mon -.->|scrapes| edge
    vault -.->|short-lived certs| apps
```

## Contents

| Doc | Topic |
| --- | --- |
| [`docs/01-overview.md`](docs/01-overview.md) | Goals, non-goals, the platform rules |
| [`docs/02-architecture.md`](docs/02-architecture.md) | C4 context and container views |
| [`docs/03-networking-and-access.md`](docs/03-networking-and-access.md) | Ingress, segmentation, admin isolation |
| [`docs/04-identity-and-authz.md`](docs/04-identity-and-authz.md) | SSO, forward-auth, authorisation model |
| [`docs/05-delivery.md`](docs/05-delivery.md) | Locked main, merge gate, split CI, digest-pinned releases, deploy loop |
| [`docs/06-data-and-backup.md`](docs/06-data-and-backup.md) | Persistence, backup, restore testing |
| [`docs/07-observability.md`](docs/07-observability.md) | Metrics, logs, alerting, uptime, security detection |
| [`docs/08-secrets-and-workload-identity.md`](docs/08-secrets-and-workload-identity.md) | Where secrets live, private CA, mTLS between machines |
| [`docs/decisions/`](docs/decisions) | Architecture decision records |
| [`docs/glossary.md`](docs/glossary.md) | Terms |

## Status

Living document. Decisions are recorded as ADRs; superseded ones are kept and
marked, not deleted.
