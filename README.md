# ZAP Proxy Docker Compose

Docker Compose configuration for running OWASP ZAP (Zed Attack Proxy) with Traefik reverse proxy.

## Features

- ZAP Stable with web-based GUI (zap-webswing)
- Automatic HTTPS with Let's Encrypt
- Traefik integration
- Persistent data storage

## Configuration

Set the following environment variable:

- `ZAP_DOMAIN`: Your domain for ZAP (e.g., `zap.yourdomain.com`)

## Deployment

1. Copy `.env.example` to `.env` and set `ZAP_DOMAIN`
2. Add DNS A record: `zap.yourdomain.com` → your server IP
3. Deploy with `docker compose up -d`

## Access

Access the ZAP web UI at: `https://${ZAP_DOMAIN}`

## Ports

- 8080: ZAP web UI (via Traefik)
- 8090: ZAP API

## Documentation

- [ZAP Docker Guide](https://www.zaproxy.org/docs/docker/about/)
- [ZAP Official Site](https://www.zaproxy.org/)
