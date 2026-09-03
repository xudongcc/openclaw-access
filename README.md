# OpenClaw Access

Local GitHub authentication for OpenClaw using Caddy and oauth2-proxy.

## Prerequisites

- OpenClaw listens on `127.0.0.1:18789`.
- Docker Desktop 4.34 or newer has **Enable host networking** enabled under
  **Settings > Resources > Network**.
- Enhanced Container Isolation is disabled because Docker Desktop host
  networking is incompatible with it.
- A GitHub OAuth App has this callback URL:
  `http://localhost:8080/oauth2/callback`.

## Start

```bash
cp .env.example .env
openssl rand -base64 32 | tr -- '+/' '-_'
```

Put the generated value and the GitHub OAuth App credentials in `.env`, then
start the stack:

```bash
docker compose config
docker compose up -d
```

Open <http://localhost:8080>.

This first-stage configuration is deliberately bound to host loopback and uses
an insecure OAuth cookie for local HTTP testing. Before exposing it through
`https://claw.kudeploy.com`, change the callback URL to HTTPS and set
`--cookie-secure=true` in `compose.yaml`.
