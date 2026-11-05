# FlareSolverr Configuration

## Overview

FlareSolverr is a proxy server to bypass Cloudflare protection on web requests. It's essential for Prowlarr and Indexers to access sites protected by Cloudflare.

**Container Name:** flaresolverr  
**Image:** ghcr.io/flaresolverr/flaresolverr:latest  
**Port:** 8191  
**Region:** Europe/London

## Container Configuration

```yaml
flaresolverr:
  image: ghcr.io/flaresolverr/flaresolverr:latest
  container_name: flaresolverr
  environment:
    - LOG_LEVEL=${LOG_LEVEL:-info}
    - LOG_FILE=${LOG_FILE:-none}
    - LOG_HTML=${LOG_HTML:-false}
    - CAPTCHA_SOLVER=${CAPTCHA_SOLVER:-none}
    - TZ=Europe/London
  ports:
    - "8191:8191"
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8191"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 60s
  volumes:
    - /var/lib/flaresolver:/config
  restart: unless-stopped
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| LOG_LEVEL | info | Log verbosity (trace, debug, info, warn, error) |
| LOG_FILE | none | Log output file path (none = stdout) |
| LOG_HTML | false | HTML logging format |
| CAPTCHA_SOLVER | none | CAPTCHA solving service (none, harvester, imagetyperz) |
| TZ | Europe/London | Timezone |

## Ports

| Port | Purpose |
|------|----------|
| 8191 | FlareSolverr API endpoint |

## Health Check

- **Test:** HTTP GET to http://localhost:8191
- **Interval:** 30 seconds
- **Timeout:** 10 seconds
- **Retries:** 3
- **Start Period:** 60 seconds

## Volumes

| Container Path | Host Path | Purpose |
|-----------------|-----------|----------|
| /config | /var/lib/flaresolver | Configuration and cache |

## Features

- Cloudflare bypass
- CAPTCHA solving (optional)
- Request proxying
- Cookie management
- JavaScript rendering
- Headless browser operation

## Integration

FlareSolverr integrates with:
- **Prowlarr** - Indexer access
- **Sonarr** - Search functionality
- **Radarr** - Search functionality
- **Custom Indexers** - Any Cloudflare-protected site

## Documentation

- GitHub: https://github.com/flaresolverr/flaresolverr
- API Docs: https://flaresolverr.com/
