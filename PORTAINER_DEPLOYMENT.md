# ZAP Proxy - Portainer Local Deployment

## Deployment Summary

Successfully deployed ZAP (Zed Attack Proxy) to Portainer at 192.168.7.10

**Stack Name:** zaproxy
**Stack ID:** 200
**Status:** Running ✓

## Access Information

### ZAP Web UI
- **URL:** http://192.168.7.10:8081
- Access the web-based GUI to configure ZAP and view captured traffic

### ZAP Proxy & API
- **Proxy Host:** 192.168.7.10
- **Proxy Port:** 8090
- Configure your browser or applications to use this as HTTP/HTTPS proxy

## Authentication

**ZAP API Key:** `043f525f251f966be97f3b7de076284ec1c8a2a69208771b4563d95d743a5983`

Use this key when making API requests to ZAP.

## Using as HTTPS MITM Proxy

### 1. Download the CA Certificate

**Via Web UI:**
1. Go to http://192.168.7.10:8081
2. Navigate to **Tools** → **Options** → **Dynamic SSL Certificates**
3. Click **Save** to download `owasp_zap_root_ca.cer`

**Via API:**
```bash
curl "http://192.168.7.10:8090/OTHER/core/other/rootcert/?apikey=043f525f251f966be97f3b7de076284ec1c8a2a69208771b4563d95d743a5983" > owasp_zap_root_ca.cer
```

### 2. Install the CA Certificate

**Linux (System-wide for Chrome/Chromium):**
```bash
# Convert to .crt format
openssl x509 -in owasp_zap_root_ca.cer -inform DER -out zap-root-ca.crt

# Install system-wide (Ubuntu/Debian)
sudo cp zap-root-ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

**Firefox:**
1. Open Firefox → Settings → Privacy & Security → Certificates → View Certificates
2. Click **Authorities** tab → **Import**
3. Select `owasp_zap_root_ca.cer`
4. Check "Trust this CA to identify websites"

**macOS:**
```bash
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain owasp_zap_root_ca.cer
```

**Windows:**
1. Double-click `owasp_zap_root_ca.cer`
2. Click **Install Certificate**
3. Select **Local Machine**
4. Choose **Trusted Root Certification Authorities**

### 3. Configure Your Browser/Application

**Browser Proxy Settings:**
- HTTP Proxy: `192.168.7.10:8090`
- HTTPS Proxy: `192.168.7.10:8090`

**Command Line Examples:**
```bash
# cURL
curl --proxy http://192.168.7.10:8090 https://example.com

# wget
wget -e use_proxy=yes -e http_proxy=192.168.7.10:8090 -e https_proxy=192.168.7.10:8090 https://example.com

# Environment variables
export http_proxy=http://192.168.7.10:8090
export https_proxy=http://192.168.7.10:8090
```

**Python requests:**
```python
proxies = {
    'http': 'http://192.168.7.10:8090',
    'https': 'http://192.168.7.10:8090',
}
response = requests.get('https://example.com', proxies=proxies)
```

### 4. Verify Setup

1. Configure your browser to use the proxy
2. Visit any HTTPS website (e.g., https://example.com)
3. Open ZAP Web UI at http://192.168.7.10:8081
4. Check the **History** tab to see captured requests

## API Usage Examples

**Get ZAP version:**
```bash
curl "http://192.168.7.10:8090/JSON/core/view/version/?apikey=043f525f251f966be97f3b7de076284ec1c8a2a69208771b4563d95d743a5983"
```

**View alerts:**
```bash
curl "http://192.168.7.10:8090/JSON/core/view/alerts/?apikey=043f525f251f966be97f3b7de076284ec1c8a2a69208771b4563d95d743a5983"
```

## Stack Management

**View in Portainer:**
http://192.168.7.10:9000/#!/3/docker/stacks/zaproxy

**Restart the stack:**
```bash
curl -X POST -H "X-API-Key: ptr_Cw1b89ICPR3IxZCubu4JvjwN7L71DrJPucmwCfbSjCA=" \
  "http://192.168.7.10:9000/api/stacks/200/stop?endpointId=3"

curl -X POST -H "X-API-Key: ptr_Cw1b89ICPR3IxZCubu4JvjwN7L71DrJPucmwCfbSjCA=" \
  "http://192.168.7.10:9000/api/stacks/200/start?endpointId=3"
```

## Data Persistence

ZAP data is stored in a Docker volume: `zaproxy_zap-data`

This includes:
- Session files
- Captured traffic
- Configuration settings
- Custom scripts

## Security Considerations

- The ZAP proxy is accessible from your local network (192.168.7.x)
- No authentication is required for proxy connections
- API operations require the API key
- Only install the CA certificate on devices you control
- Remove the CA certificate after testing to maintain security
- Consider network firewall rules if you want to restrict access further

## Troubleshooting

**Container not starting:**
```bash
# Check container logs via Portainer API
curl -H "X-API-Key: ptr_Cw1b89ICPR3IxZCubu4JvjwN7L71DrJPucmwCfbSjCA=" \
  "http://192.168.7.10:9000/api/endpoints/3/docker/containers/zaproxy-zaproxy-1/logs?stdout=1&stderr=1&tail=100"
```

**Proxy not working:**
1. Verify container is running: http://192.168.7.10:9000
2. Check port 8090 is accessible: `nc -zv 192.168.7.10 8090`
3. Ensure CA certificate is properly installed
4. Check browser proxy settings

**Web UI not accessible:**
1. Check port 8081 is accessible: `nc -zv 192.168.7.10 8081`
2. Wait for container to be fully healthy (can take 30-60 seconds)
3. Check container logs for errors

## Files

- `docker-compose.local.yml` - Portainer deployment configuration
- `docker-compose.yml` - Original Traefik/cloud deployment
- `PORTAINER_DEPLOYMENT.md` - This file
