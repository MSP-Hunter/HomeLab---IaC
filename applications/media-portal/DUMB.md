# DUMB (Debrid Unlimited Media Bridge) Configuration

## Overview

DUMB (Debrid Unlimited Media Bridge) is a powerful media streaming solution that integrates with Debrid services to provide seamless access to vast media libraries through Jellyfin and other media servers.

**Container Name:** DUMB  
**Image:** iampuid0/dumb:latest  
**Ports:** 3005, 8096, 8282, 8182  
**User:** 1000:1000  
**Timezone:** EST  
**GitHub:** https://github.com/I-am-PUID-0/DUMB

## Container Configuration

```yaml
DUMB:
  container_name: DUMB
  image: iampuid0/dumb:latest
  stop_grace_period: 30s
  shm_size: 128mb
  stdin_open: true
  tty: true
  volumes:
    - /root/docker/DUMB/config:/config
    - /root/docker/DUMB/log:/log
    - /root/docker/DUMB/data:/data
    - /root/docker/DUMB/mnt/debrid:/mnt/debrid:rshared
  environment:
    - TZ=EST
    - PUID=1000
    - PGID=1000
  ports:
    - "3005:3005"
    - "8096:8096"
    - "8282:8282"
    - "8182:8182"
  devices:
    - /dev/fuse:/dev/fuse:rwm
    - /dev/dri:/dev/dri
  cap_add:
    - SYS_ADMIN
  security_opt:
    - apparmor:unconfined
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8096"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 1200s
  restart: unless-stopped
```

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| TZ | EST | Timezone |
| PUID | 1000 | User ID for container process |
| PGID | 1000 | Group ID for container process |

## Volumes

| Container Path | Host Path | Purpose | Flags |
|-----------------|-----------|---------|-------|
| /config | /root/docker/DUMB/config | Configuration files | - |
| /log | /root/docker/DUMB/log | Application logs | - |
| /data | /root/docker/DUMB/data | Application data | - |
| /mnt/debrid | /root/docker/DUMB/mnt/debrid | Debrid mount (FUSE) | rshared |

## Ports

| Port | Purpose | Service |
|------|---------|----------|
| 3005 | API/Control | DUMB API |
| 8096 | Media Server | Jellyfin/Emby interface |
| 8282 | Cache/Metadata | Internal service |
| 8182 | Configuration | Admin interface |

## Special Requirements

### Devices

```yaml
devices:
  - /dev/fuse:/dev/fuse:rwm    # FUSE for mount operations
  - /dev/dri:/dev/dri          # GPU acceleration (optional)
```

### Capabilities

```yaml
cap_add:
  - SYS_ADMIN    # Required for FUSE mount

security_opt:
  - apparmor:unconfined    # Disable AppArmor restrictions
```

### Memory/Resources

```yaml
shm_size: 128mb    # Shared memory for FUSE operations
stop_grace_period: 30s    # Allow clean shutdown
```

### Debrid Integration

In admin panel (http://localhost:8182):
1. Settings → Debrid
2. Enter API keys for:
   - Real-Debrid
   - Alldebrid (optional)
3. Configure download preferences
4. Set cache behavior

### Mount Configuration

The FUSE mount at `/mnt/debrid` provides:
- Virtual file access to Debrid content
- Transparent caching
- Automatic cleanup

### Download Folders

DUMB creates download folders for Arr integration:
```
/mnt/debrid/decypharr_downloads/
├── sonarr/     # TV show downloads
└── radarr/     # Movie downloads
```

## Integration with Arr Stack

### Sonarr Integration

1. In Sonarr → Settings → Download Clients
2. Add custom script pointing to DUMB
3. Configure:
   - Category: Series
   - Download path: `/mnt/debrid/decypharr_downloads/sonarr`

### Radarr Integration

1. In Radarr → Settings → Download Clients
2. Add custom script pointing to DUMB
3. Configure:
   - Category: Movie
   - Download path: `/mnt/debrid/decypharr_downloads/radarr`

## Documentation

- GitHub: https://github.com/I-am-PUID-0/DUMB
- Official Site: https://dumbarr.com
- Issues/Support: https://github.com/I-am-PUID-0/DUMB/issues
