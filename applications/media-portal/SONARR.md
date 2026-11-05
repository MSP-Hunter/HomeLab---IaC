# Sonarr Configuration

## Overview

Sonarr is a PVR (Personal Video Recorder) for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new episodes of your favorite shows and will interface with clients and indexers to grab, sort, and rename them.

**Container Name:** sonarr  
**Image:** lscr.io/linuxserver/sonarr:latest  
**Port:** 8989  
**User:** 1000:1000  
**Timezone:** EST

## Container Configuration

```yaml
sonarr:
  image: lscr.io/linuxserver/sonarr:latest
  container_name: sonarr
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=EST
  volumes:
    - /root/docker/sonarr/config:/config
    - /root/docker/DUMB/mnt/debrid:/mnt/debrid
    - /root/docker/DUMB/mnt/debrid/decypharr_downloads/sonarr:/mnt/sonarr:rslave
  ports:
    - "8989:8989"
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8989"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 60s
  restart: unless-stopped
```

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| PUID | 1000 | User ID for container process |
| PGID | 1000 | Group ID for container process |
| TZ | EST | Timezone |

## Volumes

| Container Path | Host Path | Purpose | Flags |
|-----------------|-----------|---------|-------|
| /config | /root/docker/sonarr/config | Configuration and database | - |
| /mnt/debrid | /root/docker/DUMB/mnt/debrid | DUMB debrid mount | - |
| /mnt/sonarr | /root/docker/DUMB/mnt/debrid/decypharr_downloads/sonarr | Episode downloads | rslave |

## Ports

| Port | Purpose |
|------|----------|
| 8989 | Web UI access |

## Documentation

- Official: https://wiki.servarr.com/en/sonarr
- GitHub: https://github.com/Sonarr/Sonarr
- API: https://sonarr.tv/docs/api/
