# Radarr Configuration

## Overview

Radarr is a movie collection manager for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new movies and will interface with clients and indexers to grab, sort, and rename them.

**Container Name:** radarr  
**Image:** lscr.io/linuxserver/radarr:latest  
**Port:** 7878  
**User:** 1000:1000  
**Timezone:** EST

## Container Configuration

```yaml
radarr:
  image: lscr.io/linuxserver/radarr:latest
  container_name: radarr
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=EST
  volumes:
    - /root/docker/radarr/config:/config
    - /root/docker/DUMB/mnt/debrid:/mnt/debrid
    - /root/docker/DUMB/mnt/debrid/decypharr_downloads/radarr:/mnt/radarr:rslave
  ports:
    - "7878:7878"
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:7878"]
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
| /config | /root/docker/radarr/config | Configuration and database | - |
| /mnt/debrid | /root/docker/DUMB/mnt/debrid | DUMB debrid mount | - |
| /mnt/radarr | /root/docker/DUMB/mnt/debrid/decypharr_downloads/radarr | Movie downloads | rslave |

## Ports

| Port | Purpose |
|------|----------|
| 7878 | Web UI access |

## Documentation

- Official: https://wiki.servarr.com/en/radarr
- GitHub: https://github.com/Radarr/Radarr
- API: https://radarr.video/docs/api/
