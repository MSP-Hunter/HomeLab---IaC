# Wazuh — Docker Compose

This file shows a small, single-node Wazuh setup for my homelab using Docker Compose and a short note on exposing the dashboard via a reverse proxy. I keep authentication and routing in front of this with Authentik, so the compose below is intentionally small and opinion-free.

> Use proper Wazuh image tags for the version you want — these examples use `:latest` for brevity.

## What this sets up
- Wazuh indexer (search/backend)
- Wazuh manager
- Wazuh dashboard (UI)
All three run on a single Docker network. Persist data under ./data/ so containers can be recreated safely.

## docker-compose.yml example

```yaml
version: "3.8"

services:
  wazuh-indexer:
    image: wazuh/wazuh-indexer:latest
    container_name: wazuh-indexer
    restart: unless-stopped
    environment:
      - "discovery.type=single-node"
    ports:
      - "9200:9200"
    volumes:
      - ./data/indexer:/var/lib/wazuh-indexer
    networks:
      - wazuh-net

  wazuh-manager:
    image: wazuh/wazuh-manager:latest
    container_name: wazuh-manager
    restart: unless-stopped
    depends_on:
      - wazuh-indexer
    ports:
      - "1514:1514/udp"   # agent communication
      - "1515:1515"
    environment:
      - "WAZUH_INDEXER_HOSTS=http://wazuh-indexer:9200"
    volumes:
      - ./data/manager:/var/ossec
    networks:
      - wazuh-net

  wazuh-dashboard:
    image: wazuh/wazuh-dashboard:latest
    container_name: wazuh-dashboard
    restart: unless-stopped
    depends_on:
      - wazuh-indexer
      - wazuh-manager
    environment:
      - "ELASTICSEARCH_HOSTS=http://wazuh-indexer:9200"
    expose:
      - "5601"
    networks:
      - wazuh-net

networks:
  wazuh-net:
    driver: bridge
```

Notes:
- The dashboard is exposed only inside the Docker network (via `expose`). Use a reverse proxy (or another fronting service) to provide external TLS and access.
- Persist the `./data/*` folders (or use named volumes) and ensure permissions match the container user.
- Adjust ulimits, memory, and image tags according to the Wazuh documentation for production or larger deployments.

## Expose / proxy the dashboard (short)
Create a simple reverse-proxy service (nginx, Traefik, or your platform) that terminates TLS and forwards requests to the dashboard at http://wazuh-dashboard:5601. In my setup Authentik handles authentication and routing in front of the proxy, so I keep the proxy config minimal — it only needs to TLS-terminate and proxy to the dashboard. If you prefer the proxy to do authentication, add the necessary headers or integrate with your IdP, but that's outside this doc.

If you don't have DNS for the homelab, add a hosts entry on clients pointing your dashboard hostname to the Docker host IP.

## Quick troubleshooting
- docker-compose logs -f wazuh-dashboard wazuh-manager wazuh-indexer
- Ensure the dashboard can reach the indexer at the ELASTICSEARCH/OPENSEARCH host you configured.
- If agents can't connect, re-check ports (1514 UDP, 1515) and firewall rules.
