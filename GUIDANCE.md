# NetBird on Wodby

What Wodby sets up for a self-hosted NetBird server on this service. It runs the `wodby/netbird` image: NetBird's
management and signal server with its built-in sign-in, the relay and the dashboard in one container, behind nginx.
It uses NetBird's own binaries and dashboard, unchanged.

## Addresses

The service has two HTTPS addresses, and needs no published TCP or UDP ports, so it works on every cluster:

- The main domain (endpoint `netbird`) serves the dashboard, the management API, sign-in (`/oauth2`) and the gRPC
  services that NetBird clients use. The gateway reaches it over cleartext HTTP/2, which gRPC needs.
- The relay's domain (endpoint `relay`, technical host `relay.<service>.<environment domain>`) serves the relay over
  WebSocket. The server gives this address to clients; users never enter it.

Both domains are passed to the container as `NETBIRD_URL` and `NETBIRD_RELAY_URL`. After adding a custom domain to
either endpoint and making it primary, redeploy. Clients use the main domain as their management URL, so moving it
means reconnecting every client with the new URL.

## Sign in

The first deployment creates the administrator from the required settings **Admin email** and **Admin password**.
Sign in at the main domain with them. Changing the settings later has no effect; change passwords and invite users in
the dashboard. The dashboard never offers a public setup page.

NetBird's built-in sign-in keeps local users. Single sign-on providers can be added in the dashboard under
**Settings > Identity Providers**.

## Connect devices

In the dashboard, create a setup key under **Setup Keys**, or let users sign in from the client. On each device,
install the NetBird client and run:

```shell
netbird up --management-url https://<main domain>
```

Add `--setup-key <key>` for devices without a browser.

## How peers connect

Peers find their public address with Cloudflare's public STUN server (`stun:stun.cloudflare.com:3478`) and connect
to each other directly when their networks allow it. Otherwise their traffic goes through the relay, and its
bandwidth comes from the cluster. NetBird's own STUN server isn't used: behind the cluster's gateway it would see the
gateway's address instead of the peers'. Set `NETBIRD_STUN_URIS` to use other STUN servers.

## Data

The `data` volume is mounted at `/var/lib/netbird`. It holds the SQLite databases for accounts and peers, sign-in and
activity, and the GeoLite databases used by location posture checks. The first start downloads the GeoLite
databases, which can take a few minutes before the service is ready.

The generated tokens `store-encryption-key` (encrypts sensitive data in the database), `relay-secret` and
`session-cookie-key` must stay the same. Changing the store key makes the encrypted data unreadable.

Run one replica only.

## Check the result

`curl -fsS http://localhost:8080/oauth2/.well-known/openid-configuration` inside the container answers with the
sign-in configuration once the server is ready. The log shows `management server version` at start and
`WS client connected` for each relayed client.
