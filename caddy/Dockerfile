ARG CADDY_VERSION=2.11.4

FROM caddy:${CADDY_VERSION}-builder-alpine AS builder

ARG CADDY_DNS_CLOUDFLARE_VERSION=v0.2.4

RUN xcaddy build \
    --with github.com/caddy-dns/cloudflare@${CADDY_DNS_CLOUDFLARE_VERSION}

FROM caddy:${CADDY_VERSION}-alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
