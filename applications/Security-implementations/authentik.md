# authentik

A short, simple Authentik setup for Homelab using Docker Compose (Traefik-aware) and an optional Nginx reverse proxy.

## Quick setup
1. Copy the docker-compose file below to `applications/Security-implementations/docker-compose.yml`.
2. Create a `.env` file alongside it with required variables (example below).
3. Ensure you have an external Docker network `traefik-public` if you use Traefik (the compose expects it).
4. Start with `docker compose up -d` (this compose targets Docker Swarm features/labels; adapt if using plain Compose).
5. If not using Traefik, see the Nginx section for a simple proxy example.

.env example (minimal)

```env
PG_PASS=your_db_password
PG_USER=authentik
PG_DB=authentik
REDIS_PASSWORD=strong_redis_password
AUTHENTIK_SECRET_KEY=change_me_to_a_random_string
AUTHENTIK_HOST=auth.example.com
AUTHENTIK_BOOTSTRAP_PASSWORD=changeme
AUTHENTIK_EMAIL__HOST=smtp.example.com
AUTH_EMAIL_USER=smtp_user
AUTHENTIK_EMAIL__PASSWORD=smtp_password
AUTH_EMAIL_FROM=auth@example.com
```

## Docker Compose (Traefik-aware)

Use the following compose file (from nilvanlopes/authentik) as `applications/Security-implementations/docker-compose.yml`.

```yaml
version: '3.8'

services:
  postgresql:
    image: postgres:16-alpine
    volumes:
      - authentik-database:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: ${PG_PASS:?database password required}
      POSTGRES_USER: ${PG_USER:-authentik}
      POSTGRES_DB: ${PG_DB:-authentik}
      POSTGRES_INITDB_ARGS: "--auth-host=scram-sha-256"
    networks:
      - authentik-internal
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: on-failure

  redis:
    image: redis:7-alpine
    command:
      - redis-server
      - --save
      - "60"
      - "1"
      - --loglevel
      - warning
      - --requirepass
      - ${REDIS_PASSWORD}
      - --maxmemory
      - 256mb
      - --maxmemory-policy
      - allkeys-lru
    volumes:
      - authentik-redis:/data
    networks:
      - authentik-internal
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: on-failure

  server:
    image: ghcr.io/goauthentik/server:2025.8
    command: server
    user: "1000:1000"
    environment:
      AUTHENTIK_REDIS__HOST: redis
      AUTHENTIK_REDIS__PASSWORD: ${REDIS_PASSWORD}
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__USER: ${PG_USER:-authentik}
      AUTHENTIK_POSTGRESQL__NAME: ${PG_DB:-authentik}
      AUTHENTIK_POSTGRESQL__PASSWORD: ${PG_PASS}
      AUTHENTIK_SECRET_KEY: ${AUTHENTIK_SECRET_KEY}
      AUTHENTIK_ERROR_REPORTING__ENABLED: "false"
      AUTHENTIK_LOG_LEVEL: ${AUTHENTIK_LOG_LEVEL:-info}
      AUTHENTIK_COOKIE__SECURE: "true"
      AUTHENTIK_DISABLE_UPDATE_CHECK: "true"
      AUTHENTIK_BOOTSTRAP_PASSWORD: ${AUTHENTIK_BOOTSTRAP_PASSWORD}
      AUTHENTIK_EMAIL__HOST: ${AUTHENTIK_EMAIL__HOST}
      AUTHENTIK_EMAIL__PORT: 587
      AUTHENTIK_EMAIL__USERNAME: ${AUTH_EMAIL_USER}
      AUTHENTIK_EMAIL__PASSWORD: ${AUTHENTIK_EMAIL__PASSWORD}
      AUTHENTIK_EMAIL__USE_TLS: "true"
      AUTHENTIK_EMAIL__USE_SSL: "false"
      AUTHENTIK_EMAIL__FROM: ${AUTH_EMAIL_FROM}

    volumes:
      - authentik-media:/media
      - authentik-templates:/templates
    networks:
      - traefik-public
      - authentik-internal
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=traefik-public"
        
        # HTTPS Router
        - "traefik.http.routers.authentik.rule=Host(`${AUTHENTIK_HOST}`)"
        - "traefik.http.routers.authentik.entrypoints=websecure"
        - "traefik.http.routers.authentik.tls.certresolver=cloudflare"
        - "traefik.http.routers.authentik.middlewares=crowdsec@file,authentik-headers"
        - "traefik.http.routers.authentik.service=authentik"
        
        # Service
        - "traefik.http.services.authentik.loadbalancer.server.port=9000"
        
        # Security headers
        - "traefik.http.middlewares.authentik-headers.headers.stsSeconds=31536000"
        - "traefik.http.middlewares.authentik-headers.headers.stsIncludeSubdomains=true"
        - "traefik.http.middlewares.authentik-headers.headers.stsPreload=true"
        - "traefik.http.middlewares.authentik-headers.headers.forceSTSHeader=true"
        - "traefik.http.middlewares.authentik-headers.headers.frameDeny=true"
        - "traefik.http.middlewares.authentik-headers.headers.contentTypeNosniff=true"
        - "traefik.http.middlewares.authentik-headers.headers.browserXssFilter=true"
        - "traefik.http.middlewares.authentik-headers.headers.referrerPolicy=strict-origin-when-cross-origin"

  worker:
    image: ghcr.io/goauthentik/server:2025.8
    command: worker
    user: root
    environment:
      AUTHENTIK_REDIS__HOST: redis
      AUTHENTIK_REDIS__PASSWORD: ${REDIS_PASSWORD}
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__USER: ${PG_USER:-authentik}
      AUTHENTIK_POSTGRESQL__NAME: ${PG_DB:-authentik}
      AUTHENTIK_POSTGRESQL__PASSWORD: ${PG_PASS}
      AUTHENTIK_SECRET_KEY: ${AUTHENTIK_SECRET_KEY}
      AUTHENTIK_ERROR_REPORTING__ENABLED: "false"
      AUTHENTIK_LOG_LEVEL: ${AUTHENTIK_LOG_LEVEL:-info}
      AUTHENTIK_EMAIL__HOST: ${AUTHENTIK_EMAIL__HOST}
      AUTHENTIK_EMAIL__PORT: 587
      AUTHENTIK_EMAIL__USERNAME: ${AUTH_EMAIL_USER}
      AUTHENTIK_EMAIL__PASSWORD: ${AUTHENTIK_EMAIL__PASSWORD}
      AUTHENTIK_EMAIL__USE_TLS: "true"
      AUTHENTIK_EMAIL__USE_SSL: "false"
      AUTHENTIK_EMAIL__FROM: ${AUTH_EMAIL_FROM}
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - authentik-media:/media
      - authentik-certs:/certs
      - authentik-templates:/templates
    networks:
      - authentik-internal
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: on-failure

volumes:
  authentik-database:
    driver: local
  authentik-redis:
    driver: local
  authentik-media:
    driver: local
  authentik-certs:
    driver: local
  authentik-templates:
    driver: local

networks:
  traefik-public:
    external: true
  authentik-internal:
    driver: overlay
```


## Nginx reverse proxy (if not using Traefik)
If you don't run Traefik, you can proxy Authentik behind Nginx. This compose expects Authentik to listen on port 9000 inside the container (Traefik service port), or you can expose the service and point Nginx at that host/port.

A minimal HTTP -> upstream example:

```nginx
server {
    listen 80;
    server_name auth.example.com;

    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_buffering off;
        proxy_redirect off;
    }
}
```

Notes:
- The provided compose is written with Docker Swarm labels and an external Traefik network in mind—if you use plain docker-compose, remove/adjust `deploy` and Traefik labels and expose the server port.
- Keep secrets out of version control. Use a `.env` file or a secrets manager.
- Ensure `AUTHENTIK_HOST` matches your domain and your reverse proxy/TLS config.
