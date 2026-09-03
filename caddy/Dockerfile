FROM caddy:builder-alpine AS builder

ARG CADDY_DNS_CLOUDFLARE_VERSION=master

RUN xcaddy build \
    --with github.com/caddy-dns/cloudflare@${CADDY_DNS_CLOUDFLARE_VERSION}

FROM caddy:alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
