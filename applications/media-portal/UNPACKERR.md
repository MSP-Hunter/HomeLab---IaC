# Unpackerr Configuration

## Overview

Unpackerr automatically extracts downloaded files in your download folders. It's designed to work with Sonarr, Radarr, and other Arr applications to automatically extract archives after downloads complete.

**Container Name:** unpackerr  
**Image:** golift/unpackerr  
**User:** 1000:1000  
**Health Check Port:** 7878

## Container Configuration

```yaml
unpackerr:
  image: golift/unpackerr
  container_name: unpackerr
  volumes:
    - /root/docker/DUMB/mnt/debrid/decypharr_downloads/sonarr:/mnt/sonarr
    - /root/docker/DUMB/mnt/debrid/decypharr_downloads/radarr:/mnt/radarr
    - /root/docker/unpackerr/config:/config
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:7878"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 60s
  restart: always
  user: 1000:1000
  environment:
    # See configuration section below
```

## Volumes

| Container Path | Host Path | Purpose |
|-----------------|-----------|----------|
| /mnt/sonarr | /root/docker/DUMB/mnt/debrid/decypharr_downloads/sonarr | TV show downloads |
| /mnt/radarr | /root/docker/DUMB/mnt/debrid/decypharr_downloads/radarr | Movie downloads |
| /config | /root/docker/unpackerr/config | Configuration files |

## Documentation

- GitHub: https://github.com/unpackerr/unpackerr
- Configuration: https://unpackerr.zip/docs/
