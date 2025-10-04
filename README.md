# ZAP Proxy for Dokploy

Docker Compose configuration for running OWASP ZAP (Zed Attack Proxy) on Dokploy with Traefik reverse proxy.

## Features

- ZAP Stable with web-based GUI (zap-webswing)
- Automatic HTTPS with Let's Encrypt
- Traefik integration
- Persistent data storage

## Configuration

Set the following environment variable in Dokploy:

- `ZAP_DOMAIN`: Your domain for ZAP (e.g., `zap.yourdomain.com`)

## Deployment

1. Create a new Docker Compose service in Dokploy
2. Paste the contents of `docker-compose.yml`
3. Add environment variable: `ZAP_DOMAIN=zap.yourdomain.com`
4. Add DNS A record: `zap.yourdomain.com` → your server IP
5. Deploy

## Access

Access the ZAP web UI at: `https://${ZAP_DOMAIN}`

## Ports

- 8080: ZAP web UI (via Traefik)
- 8090: ZAP API

## Documentation

- [ZAP Docker Guide](https://www.zaproxy.org/docs/docker/about/)
- [ZAP Official Site](https://www.zaproxy.org/)
