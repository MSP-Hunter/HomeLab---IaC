# Homelab Pi-hole DNS Setup

A self-hosted DNS filtering setup using Pi-hole on Proxmox LXC, with Cloudflare Tunnel integration, static IP infrastructure, and network-wide ad/tracker blocking via Hagezi blocklists.

---

## Overview

- **Pi-hole** running in a Proxmox LXC container
- **Static IP** assigned to the Pi-hole container for DNS stability
- **Cloudflare Tunnels** used to expose services externally — these resolve via Cloudflare's public DNS and are unaffected by local DNS changes
- **Fallback DNS** configured at every layer to prevent outages during maintenance

---

## Infrastructure Notes

### Proxmox-Managed Configuration

Proxmox automatically manages `/etc/hosts` and `/etc/resolv.conf` inside LXC containers. Both files contain a `BEGIN PVE / END PVE` block that **can be overwritten** by Proxmox during container operations (restores, migrations, template updates).

**Rules:**
- Never place custom entries inside the PVE block
- Place manual `/etc/hosts` entries **below** the PVE block
- Do **not** manually edit `/etc/resolv.conf` inside containers — change DNS at the Proxmox level instead

### Setting DNS on LXC Containers (Persistent)

```bash
# On the Proxmox host
pct set <vmid> --nameserver <pihole-ip>

# Verify inside the container
cat /etc/resolv.conf
```

This survives container restarts and Proxmox operations, unlike editing `resolv.conf` directly.

---

## DNS Configuration

### Recommended resolv.conf Structure

```
search yourdomain.local
nameserver <pihole-ip>        # Primary — Pi-hole
nameserver 1.1.1.1            # Fallback — Cloudflare
nameserver 8.8.8.8            # Fallback — Google
```

Fallback resolvers ensure name resolution continues if Pi-hole is temporarily unavailable (reboots, updates, maintenance).

### Setting DNS on Your Gateway/Router

Changing DNS on your router does **not** affect connectivity — DNS only resolves hostnames to IPs and has no involvement in routing or static IP assignments.

**What happens when you change it:**
- DHCP clients receive the new DNS on their next lease renewal
- Static IP devices are **unaffected** — they ignore DHCP DNS advertisements and must be updated individually via `pct set`

**Safe rollout order:**
1. Update gateway DNS to point to Pi-hole
2. Verify Pi-hole is receiving queries (check the dashboard query log)
3. Update containers one at a time via `pct set`
4. Test each with `dig google.com` after switching

### Pi-hole Upstream Resolvers

Configure at least two upstream resolvers inside Pi-hole itself (Settings → DNS):

| Resolver | IP |
|---|---|
| Cloudflare | `1.1.1.1` |
| Quad9 | `9.9.9.9` |

---

## Static /etc/hosts Entries

Add these **below** the PVE block on each machine so critical hosts resolve even when DNS is down.

Each machine should know about the other hosts it depends on:

```
# --- END PVE ---

# Static homelab entries - DNS fallback
<ip-of-host>    hostname.domain hostname
```

**What to include:**
- Pi-hole host
- Proxmox host (web UI access)
- Media server
- Router/gateway (if referenced by name in scripts)
- Any containers with inter-service dependencies

---

## Verifying DNS

```bash
# What DNS server is the machine using
cat /etc/resolv.conf

# Test resolution (which server answered)
dig google.com

# Test Pi-hole directly
dig google.com @<pihole-ip>

# Check what a hostname resolves to (respects /etc/hosts order)
getent hosts <hostname>

# Confirm resolution order (should show: files dns)
cat /etc/nsswitch.conf | grep hosts

# Confirm Pi-hole is listening on port 53
ss -ulnp | grep :53
```

---

## Blocklists

Managed via **[Hagezi DNS Blocklists](https://github.com/hagezi/dns-blocklists)** — added through Pi-hole's Adlists UI and activated with `pihole -g`.

### Active Lists

| List | URL | Purpose |
|---|---|---|
| Multi Normal | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/multi.txt` | General ads, trackers, telemetry |
| Multi Pro | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.txt` | Extended blocking, more aggressive |

### Adding a New Adlist

1. Pi-hole UI → **Adlists** → paste URL → **Add**
2. Run gravity update to pull and activate:

```bash
pihole -g
# or if running in Docker:
docker exec pihole pihole -g
```

### Custom Domain Blocks

Individual telemetry/tracking domains blocked manually via Pi-hole's blacklist:

| Domain | Reason |
|---|---|
| `mobile.logs.roku.com` | Roku device telemetry |
| `logs.roku.com` | Roku general logging |
| `scribe.roku.com` | Roku event tracking |

**General pattern — safe to block:**

| Subdomain Pattern | Type |
|---|---|
| `logs.*` | Logging endpoint |
| `telemetry.*` | Usage telemetry |
| `metrics.*` | Metrics collection |
| `analytics.*` | Analytics |
| `scribe.*` | Log aggregation pipeline (naming convention from Facebook's Scribe) |

**Do not block — breaks functionality:**

| Subdomain Pattern | Type |
|---|---|
| `api.*` | Device/service API |
| `auth.*` | Authentication |
| `cdn.*` | Content delivery |
| `update.*` | Firmware/software updates |
| `stream.*` / `hls.*` | Media streaming |

---

## Cloudflare Tunnel Compatibility

Services exposed via Cloudflare Tunnels (e.g. `service.yourdomain.com`) resolve through Cloudflare's public DNS. These are **unaffected** by local Pi-hole availability — if Pi-hole is down and your fallback DNS (`1.1.1.1`) kicks in, tunneled services remain accessible.

Local-only hostnames (`.local`, `.lan`, custom internal domains) are the only ones that depend on Pi-hole being up.

---

## High Availability (Optional)

For critical uptime, run a second Pi-hole instance (Docker container on a separate host) and configure it as the secondary DNS nameserver. This keeps full Pi-hole-level filtering active even during maintenance on the primary.

```
nameserver <pihole-primary-ip>
nameserver <pihole-secondary-ip>
nameserver 1.1.1.1
```

---

## References

- [Pi-hole Documentation](https://docs.pi-hole.net)
- [Hagezi DNS Blocklists](https://github.com/hagezi/dns-blocklists)
- [Proxmox LXC Documentation](https://pve.proxmox.com/wiki/Linux_Container)
- [Cloudflare Tunnel Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- [Pi-hole WireGuard VPN Concept](https://docs.pi-hole.net/guides/vpn/wireguard/concept/)
