# 2 · Architecture

The views below follow the [C4 model](https://c4model.com/): context first, then
one level down into containers. Deployment specifics (addresses, counts, sizing)
are intentionally absent — this is the shape, not an inventory.

## Context

```mermaid
flowchart TB
    user([User<br/>public internet])
    operator([Operator])
    dns[DNS + CDN/proxy<br/>external]
    offsite[(Offsite backup<br/>repository)]
    upstream[Upstream registries<br/>&amp; update feeds]

    platform[["Homelab Platform<br/>(single node)"]]

    user -->|HTTPS| dns --> platform
    operator -->|VPN| platform
    platform -->|encrypted snapshots| offsite
    platform -->|pull images / check updates| upstream
```

The platform trusts: the operator over VPN, and the git repository. It does not
trust: the public internet, an authenticated user's device, or an image tag it
has not pinned.

## Containers

```mermaid
flowchart TB
    subgraph node [Single node]
        direction TB

        subgraph edgenet [edge network]
            proxy[Reverse proxy]
            idp[Identity provider]
        end

        subgraph appnet [app network]
            app1[App A · public]
            app2[App B · SSO-gated]
            app3[App C · SSO-gated]
        end

        subgraph datanet [data network · no egress]
            db[(PostgreSQL)]
            cache[(Redis)]
            files[(Object store)]
        end

        subgraph adminnet [admin network]
            portainer[Container management]
            dash[Status dashboard]
        end

        subgraph opsnet [ops]
            deployer[Deploy agent<br/>watches git]
            backup[Backup agent]
            monitor[Metrics + alerts]
        end
    end

    repo[(Git repository)]
    vpn{{VPN endpoint}}

    proxy --> app1
    proxy --> app2
    proxy --> app3
    proxy <--> idp
    app2 --> db
    app3 --> db
    app3 --> cache
    app3 --> files
    deployer -->|pull + apply| repo
    deployer -.->|docker compose up| node
    backup -.->|snapshot data root| files
    backup -.->|dump| db
    monitor -.->|scrape| proxy
    monitor -.->|scrape| app2
    operator([Operator]) --> vpn --> adminnet
```

### Planes

- **Edge plane** — the reverse proxy and the identity provider. The only
  containers that receive traffic that originated outside the node.
- **Application plane** — the services themselves, split by whether they are
  public or require SSO. Apps never talk to each other directly; if they need
  to, they go through the edge or a defined API.
- **Data plane** — databases and storage, on a network with **no outbound
  route**. Reachable only from the specific apps that own the data.
- **Admin plane** — management UIs. On their own network, never routed by the
  public edge, reachable only over VPN. See ADR&nbsp;0003.
- **Ops plane** — the deploy agent, backup agent, and monitoring. On-node
  automation with narrowly scoped permissions.

## Request path for a gated app

```mermaid
sequenceDiagram
    participant U as User
    participant P as Reverse proxy
    participant I as Identity provider
    participant A as App

    U->>P: GET /app
    P->>I: forward-auth check (is this session valid?)
    alt no valid session
        I-->>P: 401
        P-->>U: redirect to sign-in
        U->>I: authenticate (password + TOTP)
        I-->>U: session cookie
        U->>P: GET /app (retry)
        P->>I: forward-auth check
    end
    I-->>P: 200 + identity headers
    P->>A: proxied request + identity headers
    A-->>U: response
```

The app receives the request only after the proxy has confirmed a valid session.
The app reads the identity from trusted headers set by the proxy; it never sees
credentials and has no login page of its own.
