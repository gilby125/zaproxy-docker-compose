# ZAP Proxy Docker Compose

Complete OWASP ZAP security testing stack with web UI and AI agent integration.

## Components

```
┌─────────────────────────────────────────────────────────────┐
│                    dokploy-network                          │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐ │
│  │   ZAP       │    │ MCP Server  │    │ Web Interface   │ │
│  │  (daemon)   │◄───│             │◄───│ (React UI)      │ │
│  │  :8080      │    │  :7456      │    │  :3001          │ │
│  └─────────────┘    └─────────────┘    └─────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

| Component | Directory | Port | Purpose |
|-----------|-----------|------|---------|
| ZAP Daemon | `./` (root) | 8080 | Security scanner engine |
| MCP Server | `./mcp-server/` | 7456 | AI agent integration (Claude Code, etc.) |
| Web Interface | `./web-interface/` | 3001 | Visual UI for scans & reports |

## Features

- ZAP Stable in daemon mode
- Automatic HTTPS with Let's Encrypt
- Traefik integration
- MCP server for AI-assisted security testing
- React web interface with PDF reports
- Persistent data storage

## Configuration

Set the following environment variable:

- `ZAP_DOMAIN`: Your domain for ZAP (e.g., `zap.yourdomain.com`)

## Deployment

1. Copy `.env.example` to `.env` and configure:
   ```bash
   cp .env.example .env

   # Generate a secure API key
   openssl rand -hex 32

   # Edit .env and set:
   # - ZAP_DOMAIN: Your domain
   # - ZAP_API_KEY: The generated key
   # - ADMIN_IP_WHITELIST: Your allowed IPs
   ```
2. Add DNS A record: `zap.yourdomain.com` → your server IP
3. Deploy with `docker compose up -d`

## Access

Access the ZAP web UI at: `https://${ZAP_DOMAIN}`

## Ports

- 8080: ZAP web UI (via Traefik)
- 8090: ZAP Proxy & API

## Authentication & Security

### API Key Authentication

ZAP uses an API key to authenticate API operations and can provide additional security:

**Setting the API Key:**
1. Generate a secure key: `openssl rand -hex 32`
2. Add to `.env`: `ZAP_API_KEY=your-generated-key`
3. Restart ZAP: `docker compose restart zaproxy`

**Using the API Key:**

When making API requests:
```bash
# Include API key in URL
curl "http://localhost:8090/JSON/core/view/version/?apikey=YOUR_API_KEY"

# Or download CA cert with API key
curl "http://localhost:8090/OTHER/core/other/rootcert/?apikey=YOUR_API_KEY" > zap-cert.cer
```

**Note:** The ZAP proxy itself (port 8090) doesn't enforce authentication for proxy connections. The API key is for API operations only.

### Securing the Proxy Port

Since the proxy port (8090) doesn't have built-in authentication, consider these options:

**Option 1: SSH Tunnel (Recommended for Remote Access)**
```bash
# On your local machine, create tunnel to server
ssh -L 8090:localhost:8090 user@your-server

# Then configure proxy to use localhost:8090
# This encrypts and authenticates all proxy traffic
```

**Option 2: Restrict to Localhost Only**

Modify `docker-compose.yml` to bind only to localhost:
```yaml
ports:
  - "127.0.0.1:8090:8090"  # Only accessible locally
```

Then use SSH tunnel for remote access.

**Option 3: Firewall Rules**
```bash
# Allow only specific IPs to access port 8090
sudo ufw allow from YOUR_IP_ADDRESS to any port 8090
sudo ufw deny 8090
```

**Option 4: VPN/Tailscale Only**
- Don't expose port 8090 publicly
- Access only via VPN (your Tailscale network: 100.0.0.0/8)

## Using ZAP as HTTPS MITM Proxy

### 1. Download ZAP CA Certificate

Access your ZAP instance at `https://${ZAP_DOMAIN}` and:

1. Navigate to **Tools** → **Options** → **Dynamic SSL Certificates**
2. Click **Save** to download `owasp_zap_root_ca.cer`

Alternatively, download via API:
```bash
# Without API key
curl -k "http://localhost:8090/OTHER/core/other/rootcert/" > owasp_zap_root_ca.cer

# Or with API key (if configured)
curl "http://localhost:8090/OTHER/core/other/rootcert/?apikey=YOUR_API_KEY" > owasp_zap_root_ca.cer
```

### 2. Install CA Certificate

#### Linux (Chrome/Chromium)
```bash
# Convert to .crt format
openssl x509 -in owasp_zap_root_ca.cer -inform DER -out zap-root-ca.crt

# Install system-wide (Ubuntu/Debian)
sudo cp zap-root-ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# Or import in Chrome: Settings → Privacy and security → Security → Manage certificates
```

#### Linux (Firefox)
1. Open Firefox → Settings → Privacy & Security → Certificates → View Certificates
2. Click **Authorities** tab → **Import**
3. Select `owasp_zap_root_ca.cer`
4. Check "Trust this CA to identify websites"

#### macOS
```bash
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain owasp_zap_root_ca.cer
```

#### Windows
1. Double-click `owasp_zap_root_ca.cer`
2. Click **Install Certificate**
3. Select **Local Machine**
4. Choose **Place all certificates in the following store** → **Trusted Root Certification Authorities**

### 3. Configure Proxy Settings

#### Browser Configuration
Set your browser to use proxy:
- **HTTP Proxy**: `localhost` or your server IP
- **Port**: `8090`
- **HTTPS Proxy**: `localhost` or your server IP
- **Port**: `8090`

#### Command Line Examples
```bash
# cURL
curl --proxy http://localhost:8090 https://example.com

# wget
wget -e use_proxy=yes -e http_proxy=localhost:8090 -e https_proxy=localhost:8090 https://example.com

# Environment variables
export http_proxy=http://localhost:8090
export https_proxy=http://localhost:8090
```

#### Application Configuration
For applications, set system proxy or use:
```bash
# Python requests
proxies = {
    'http': 'http://localhost:8090',
    'https': 'http://localhost:8090',
}
```

### 4. Remote Access
If ZAP is running on a remote server, you can:

1. **SSH Tunnel** (Recommended):
   ```bash
   ssh -L 8090:localhost:8090 user@your-server
   ```
   Then use `localhost:8090` as proxy

2. **Direct Connection**:
   - Ensure firewall allows port 8090
   - Use server IP: `server-ip:8090`

### 5. Verify Setup
1. Configure proxy in browser
2. Visit `https://example.com`
3. Check ZAP History tab for captured requests

## Security Best Practices

- **Web UI**: IP-restricted via Traefik (see `.env` for `ADMIN_IP_WHITELIST`)
- **API**: Protected with API key (set `ZAP_API_KEY` in `.env`)
- **Proxy Port (8090)**: Consider one of these security measures:
  - Bind to localhost only and use SSH tunnel
  - Use firewall rules to restrict access
  - Access only via VPN/Tailscale
  - Keep public but monitor access logs
- **CA Certificate**: Only install on systems you control
- **After Testing**: Remove CA certificate and revoke any temporary access

## Documentation

- [ZAP Docker Guide](https://www.zaproxy.org/docs/docker/about/)
- [ZAP Official Site](https://www.zaproxy.org/)
- [ZAP Proxy Configuration](https://www.zaproxy.org/docs/desktop/start/proxies/)
