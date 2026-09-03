# OpenClaw Access

Local GitHub authentication for OpenClaw using Caddy and oauth2-proxy.

## Prerequisites

- OpenClaw listens on `127.0.0.1:18789`.
- Docker Desktop provides `host.docker.internal` (enabled by default).
- A GitHub OAuth App has this callback URL:
  `http://192.168.28.154:8080/oauth2/callback`.

## Start

```bash
cp .env.example .env
openssl rand -base64 32 | tr -- '+/' '-_'
```

Put the generated value and the GitHub OAuth App credentials in `.env`, then
adjust the public URL, listener, or private Docker network values if needed and
start the stack:

```bash
docker compose config
docker compose up -d
```

Open <http://192.168.28.154:8080> from the local network. Caddy listens on port
`8080`; oauth2-proxy is available only on the private Compose network and
OpenClaw remains bound to host loopback.

MCP Apps and dashboard widgets use the dedicated, unauthenticated sandbox
origin <http://192.168.28.154:8081>, which proxies only to OpenClaw's sandbox
listener on port `18790`.

This deployment uses plain HTTP. Use it only on a trusted local network because
browser traffic and OAuth session cookies are not protected by TLS.
