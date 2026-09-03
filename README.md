# OpenClaw Access

Local GitHub authentication for OpenClaw using Caddy and oauth2-proxy.

## Prerequisites

- OpenClaw listens on `127.0.0.1:18789`.
- Docker Desktop provides `host.docker.internal` (enabled by default).
- A GitHub OAuth App has this callback URL:
  `https://claw.kudeploy.com/oauth2/callback`.
- A Cloudflare API token scoped to `kudeploy.com` has `Zone:Read` and
  `DNS:Edit` permissions.

## Start

```bash
cp .env.example .env
openssl rand -base64 32 | tr -- '+/' '-_'
```

Put the generated value, the GitHub OAuth App credentials, and the Cloudflare
API token in `.env`, then start the stack:

`OAUTH2_PROXY_EMAIL_DOMAINS=*` permits any email domain; narrow it when needed.
Leave `OAUTH2_PROXY_OIDC_ISSUER_URL` empty for GitHub, or set it when changing
`OAUTH2_PROXY_PROVIDER` to `oidc`.

```bash
docker compose config
docker compose up -d
```

Open <https://claw.kudeploy.com> from the local network. Caddy terminates HTTPS
on port `443`; oauth2-proxy is available only on the private Compose network
and OpenClaw remains bound to host loopback.

MCP Apps and dashboard widgets use the dedicated, unauthenticated sandbox
origin <https://claw-mcp-apps.kudeploy.com>, which proxies only to OpenClaw's sandbox
listener on port `18790`.

Certificates are issued and renewed through the Cloudflare DNS challenge.
