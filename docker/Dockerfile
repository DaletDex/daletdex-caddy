# Rebuild stock Caddy 2.11.4 with a current Go toolchain so stdlib/x/net/x/text
# High CVEs in the official prebuilt binary are patched. No extra modules
# (production Caddyfile only uses reverse_proxy + encode + ACME).
FROM golang:1.26-alpine AS build

RUN apk add --no-cache git ca-certificates

# Official caddy:2.11-alpine still ships Go 1.26.3. Force a patched toolchain.
ENV CGO_ENABLED=0 \
    GOTOOLCHAIN=go1.26.6
ARG CADDY_VERSION=v2.11.4

RUN go install github.com/caddyserver/xcaddy/cmd/xcaddy@v0.4.5
RUN /go/bin/xcaddy build "${CADDY_VERSION}" \
      --output /usr/bin/caddy \
      --replace golang.org/x/net=golang.org/x/net@v0.56.0 \
      --replace golang.org/x/text=golang.org/x/text@v0.39.0 \
      --replace google.golang.org/grpc=google.golang.org/grpc@v1.82.1 \
    && /usr/bin/caddy version

FROM alpine:3.23

ENV XDG_CONFIG_HOME=/config \
    XDG_DATA_HOME=/data

RUN apk add --no-cache ca-certificates libcap \
    && apk upgrade --no-cache \
    && mkdir -p /etc/caddy /data /config \
    && rm -rf /var/cache/apk/*

COPY --from=build /usr/bin/caddy /usr/bin/caddy
RUN setcap cap_net_bind_service=+ep /usr/bin/caddy

COPY docker/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Config smoke: official adapter + the same reverse_proxy pattern used in orch.
RUN printf 'example.com {\n\tencode zstd gzip\n\treverse_proxy frontend:80\n}\n' > /tmp/Caddyfile \
    && caddy validate --config /tmp/Caddyfile --adapter caddyfile \
    && rm -f /tmp/Caddyfile

EXPOSE 80 443

ENTRYPOINT ["/entrypoint.sh"]
