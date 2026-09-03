# OpenClaw Access

An HTTPS access layer for a locally hosted OpenClaw gateway. Caddy terminates
TLS, oauth2-proxy authenticates users, and OpenClaw receives the verified email
address through trusted-proxy authentication.

The stack exposes two origins:

- `OPENCLAW_DOMAIN`: the authenticated OpenClaw Web UI and gateway.
- `OPENCLAW_MCP_APPS_DOMAIN`: the dedicated MCP Apps sandbox origin.

## Architecture

```text
Browser -> Caddy -> oauth2-proxy -> OpenClaw gateway (127.0.0.1:18789)
        -> Caddy ----------------> MCP Apps sandbox (127.0.0.1:18790)
```

OpenClaw and its sandbox listener remain bound to the host loopback interface.
The containers reach them through Docker Desktop's `host.docker.internal`.

## Requirements

- Docker Desktop with Docker Compose.
- OpenClaw listening on `127.0.0.1:18789`.
- Two DNS records pointing to the Docker host, one for each public origin.
- A Cloudflare API token with `Zone:Read` and `DNS:Edit` for the relevant zone.
- An OAuth client. GitHub is the default; a generic OIDC provider can be used.
- Inbound TCP ports `80` and `443`, plus UDP `443` for HTTP/3 if desired.

## OAuth client

For GitHub, create an OAuth App and set its authorization callback URL to:

```text
https://<OPENCLAW_DOMAIN>/oauth2/callback
```

Use the same callback URL when configuring another OIDC provider.

## Configure

Copy the example file:

```bash
cp .env.example .env
```

Generate the oauth2-proxy cookie secret:

```bash
openssl rand -base64 32 | tr -- '+/' '-_'
```

Fill in `.env`:

```dotenv
OAUTH2_PROXY_PROVIDER=github
OAUTH2_PROXY_OIDC_ISSUER_URL=
OAUTH2_PROXY_CLIENT_ID=<oauth-client-id>
OAUTH2_PROXY_CLIENT_SECRET=<oauth-client-secret>
OAUTH2_PROXY_EMAIL_DOMAINS=*
OAUTH2_PROXY_COOKIE_SECRET=<generated-cookie-secret>

CLOUDFLARE_API_TOKEN=<cloudflare-api-token>

OPENCLAW_DOMAIN=claw.example.com
OPENCLAW_MCP_APPS_DOMAIN=claw-mcp-apps.example.com
```

`OAUTH2_PROXY_EMAIL_DOMAINS=*` accepts authenticated users from any email
domain. Restrict this value if oauth2-proxy should enforce a narrower policy.
OpenClaw should remain the authoritative user allowlist.

For OIDC, set `OAUTH2_PROXY_PROVIDER=oidc` and provide the issuer URL in
`OAUTH2_PROXY_OIDC_ISSUER_URL`. Leave the issuer URL empty for GitHub.

## Configure OpenClaw

The corresponding parts of `~/.openclaw/openclaw.json` should resemble:

```json
{
  "gateway": {
    "bind": "loopback",
    "trustedProxies": ["127.0.0.1", "::1"],
    "auth": {
      "mode": "trusted-proxy",
      "trustedProxy": {
        "userHeader": "x-auth-request-email",
        "allowUsers": ["you@example.com"]
      }
    }
  },
  "mcp": {
    "apps": {
      "enabled": true,
      "sandboxOrigin": "https://claw-mcp-apps.example.com"
    }
  }
}
```

Keep `gateway.trustedProxies` narrow. Caddy removes incoming identity headers
and forwards only the email verified by oauth2-proxy. User authorization and
administrator scopes should be configured in OpenClaw, not oauth2-proxy.

The MCP Apps origin is intentionally separate and is not routed through
oauth2-proxy. It only proxies to OpenClaw's sandbox listener. Do not route the
main gateway through this hostname.

Restart OpenClaw after changing its configuration:

```bash
openclaw config validate
openclaw gateway restart
```

## Start

Validate and start the stack:

```bash
docker compose config --quiet
docker compose up -d --build
docker compose ps
```

Caddy obtains and renews certificates using the Cloudflare DNS challenge.

## Verify

The main origin should redirect an unauthenticated request to oauth2-proxy:

```bash
curl -I "https://${OPENCLAW_DOMAIN}/"
```

The MCP Apps origin should reach the sandbox listener:

```bash
curl -I "https://${OPENCLAW_MCP_APPS_DOMAIN}/mcp-app-sandbox"
```

Inspect service logs if either check fails:

```bash
docker compose logs --tail=100 caddy oauth2-proxy
```

## Update and stop

```bash
docker compose pull
docker compose build --pull caddy
docker compose up -d
```

```bash
docker compose down
```

The named Caddy volumes retain certificates and state across container
recreation. Add `--volumes` only when that state should also be deleted.

## Security notes

- Never commit `.env`; it is ignored by this repository.
- Keep the Cloudflare token scoped to the required zone and permissions.
- Restrict OpenClaw users with `gateway.auth.trustedProxy.allowUsers`.
- Keep OpenClaw bound to loopback and expose it only through Caddy.
- The MCP Apps hostname is a sandbox content origin, not a second gateway URL.
