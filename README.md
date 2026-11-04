# NetBird service for Kubernetes on Wodby

Run a self-hosted [NetBird](https://netbird.io) server, a WireGuard-based overlay network, as a reusable Kubernetes
application service with Wodby.

This repository defines the Wodby service manifests and operational
configuration for NetBird.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [NetBird application stack](https://github.com/wodby/stack-netbird)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `netbird` |
| Type | Application service |
| Versions | `0` by default |
| Workloads | `main` (Statefulset, primary) |
| Containers | `netbird` using `wodby/netbird` |
| Endpoints | `netbird`: HTTP 8080 (main; dashboard, API, sign-in and gRPC); `relay`: HTTP 8082 (relay over WebSocket) |
| Service links | None |
| Application build | Not buildable from application source |
| Helm | chart `oci://registry-1.docker.io/wodby/stateful`; version `0.3.0` |
| Configuration and operations | 1 volume, 2 settings |

## Connect devices

1. Set **Admin email** and **Admin password** when you create the app; the first deployment creates that
   administrator.
2. Sign in to the dashboard at the service's main domain and create a setup key, or let users sign in from the
   client.
3. On each device, install the NetBird client and run `netbird up --management-url https://<main domain>`.

Everything runs over HTTPS, so no ports need publishing and it works on every cluster. Peers connect directly when
their networks allow it and through the relay otherwise.

## Use this service

Use this service through [NetBird application stack](https://github.com/wodby/stack-netbird), or reference
`netbird` from a custom Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id> --include GUIDANCE.md
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
