### Service Definition
```yaml
uptime-kuma:
  image: louislam/uptime-kuma:2              # Version 2 (latest stable)
  container_name: uptime-kuma                # Fixed container name
  restart: always                            # Always restart if crashes
  ports:
    - "3001:3001"                            # Host:Container port mapping
```

### Port Mapping
```yaml
ports:
  - "3001:3001"
```
- **Host Port**: 3001 (your machine's IP)
- **Container Port**: 3001 (Uptime Kuma inside container)
- **Access**: http://localhost:3001 or http://[your-ip]:3001

### Persistent Storage
```yaml
volumes:
  - ~/home/root/docker-kuma:/app/data       # Persistent data directory
```

### Environment Variables
```yaml
environment:
  - TZ=UTC                                   # Timezone for timestamps
  - UMASK=0022                               # File permission mask (744)
```

**Timezone Options**:
```bash
# Common timezones:
TZ=America/New_York          # Eastern Time
TZ=America/Chicago           # Central Time
TZ=America/Denver            # Mountain Time
TZ=America/Los_Angeles       # Pacific Time
TZ=Europe/London             # GMT
TZ=Europe/Paris              # CET
TZ=Asia/Tokyo                # JST
TZ=Australia/Sydney          # AEDT
```

### Core Functionality

```
┌─────────────────────────────────────────────────────┐
│         Uptime Kuma Monitoring System                │
├─────────────────────────────────────────────────────┤
│                                                      │
│  1. You Create Monitors                             │
│     └─ Add website URLs or services to monitor     │
│     └─ Configure check intervals (30s, 1m, 5m)    │
│     └─ Set timeout and retry policies              │
│                                                      │
│  2. Kuma Checks Service Health                      │
│     └─ Sends HTTP request to service               │
│     └─ Measures response time                       │
│     └─ Records UP or DOWN status                    │
│                                                      │
│  3. Kuma Tracks Uptime                              │
│     └─ Calculates availability percentage           │
│     └─ Records incident times                       │
│     └─ Maintains historical data                    │
│                                                      │
│  4. Kuma Sends Alerts                               │
│     └─ Triggers when service goes DOWN              │
│     └─ Sends to Email, Slack, Discord, etc.        │
│     └─ Sends recovery notification when UP          │
│                                                      │
│  5. Displays Dashboard                              │
│     └─ Real-time status of all monitors             │
│     └─ Historical uptime charts                     │
│     └─ Incident log                                 │
│     └─ Public status page (optional)                │
│                                                      │
└─────────────────────────────────────────────────────┘
```

