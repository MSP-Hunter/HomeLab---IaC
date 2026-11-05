# Prowlarr Configuration

## Overview

Prowlarr is a indexer manager/proxy built on the popular arr net base stack to integrate with your various PVR apps. It supports management of both Torrent Trackers and Usenet Indexers.

**Container Name:** prowlarr  
**Image:** lscr.io/linuxserver/prowlarr:latest  
**Port:** 9696  
**User:** 1000:1000  
**Timezone:** EST

## Container Configuration

```yaml
prowlarr:
  image: lscr.io/linuxserver/prowlarr:latest
  container_name: prowlarr
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=EST
  volumes:
    - /root/docker/prowlarr/config:/config
  ports:
    - "9696:9696"
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:9696"]
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

| Container Path | Host Path | Purpose |
|-----------------|-----------|----------|
| /config | /root/docker/prowlarr/config | Configuration and database |

## Ports

| Port | Purpose |
|------|----------|
| 9696 | Web UI access |

## Documentation

- Official: https://wiki.servarr.com/prowlarr
- GitHub: https://github.com/Prowlarr/Prowlarr
- API: https://prowlarr.com/docs/api/
