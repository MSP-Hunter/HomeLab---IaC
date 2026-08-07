# authentik

A short, simple Authentik setup for Homelab using Docker Compose and an optional Nginx reverse proxy.

## Quick setup
1. Copy the docker-compose file below to `applications/Security-implementations/docker-compose.authentik.yml` (or keep it next to your other compose files).
2. Edit the environment values: set a strong `AUTHENTIK_SECRET_KEY`, `POSTGRES_PASSWORD`, and `AUTHENTIK_HOST` to your domain (e.g. `auth.example.com`).
3. Start: `docker-compose -f docker-compose.authentik.yml up -d`
4. Run migrations and create an admin account inside the running container:

```
# run migrations
docker-compose -f docker-compose.authentik.yml exec authentik authentik-server migrate

# create admin (replace values)
docker-compose -f docker-compose.authentik.yml exec authentik authentik-server createadmin --username admin --email admin@example.com
```

## Example docker-compose (minimal)

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: authentik
      POSTGRES_USER: authentik
      POSTGRES_PASSWORD: authentikpassword
    volumes:
      - authentik-db:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    volumes:
      - authentik-redis:/data

  authentik:
    image: ghcr.io/goauthentik/server:latest
    restart: unless-stopped
    depends_on:
      - postgres
      - redis
    environment:
      # replace these values
      AUTHENTIK_SECRET_KEY: "change_me_to_a_random_string"
      DATABASE_URL: "postgresql://authentik:authentikpassword@postgres:5432/authentik"
      REDIS_URL: "redis://redis:6379/0"
      AUTHENTIK_HOST: "auth.example.com"
    ports:
      - "8000:8000"
    volumes:
      - authentik-data:/data

volumes:
  authentik-db:
  authentik-redis:
  authentik-data:
```

Notes:
- Keep secrets out of version control. Consider using an env file referenced by docker-compose or a secret manager.
- The container exposes port 8000 by default; we recommend proxying it behind Nginx and terminating TLS there.

## Nginx reverse proxy (simple)
Place this in your Nginx site config for the `AUTHENTIK_HOST` domain. This assumes Authentik is reachable on localhost:8000 from your proxy host.

```nginx
server {
    listen 80;
    server_name auth.example.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_buffering off;
        proxy_redirect off;
    }
}
```

Recommended additions:
- Use Certbot or another ACME client to obtain TLS and listen on 443 with `ssl_certificate` / `ssl_certificate_key`.
- Ensure `AUTHENTIK_HOST` matches the Nginx `server_name` and configure any trusted proxy settings if required.

That's it — a minimal Authentik compose and a simple Nginx proxy to get you started. Adjust images/versions and scaling (workers, outposts) as needed for production.
