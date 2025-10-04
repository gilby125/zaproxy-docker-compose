# ZAP Proxy for Dokploy

Docker Compose configuration for running OWASP ZAP (Zed Attack Proxy) on Dokploy with Traefik reverse proxy.

## Features

- ZAP Stable with web-based GUI (zap-webswing)
- Automatic HTTPS with Let's Encrypt
- Traefik integration
- Persistent data storage

## Deployment

1. Create a new Docker Compose service in Dokploy
2. Paste the contents of `docker-compose.yml`
3. Add DNS A record: `zap.throughfire.net` → your server IP
4. Deploy

## Access

Access the ZAP web UI at: https://zap.throughfire.net

## Ports

- 8080: ZAP web UI (via Traefik)
- 8090: ZAP API

## Documentation

- [ZAP Docker Guide](https://www.zaproxy.org/docs/docker/about/)
- [ZAP Official Site](https://www.zaproxy.org/)
