<p align="center"><a href="https://freedium.cfd" target="_blank"><img src="https://avatars.githubusercontent.com/u/142643505?s=200&v=4" width="20%"></a></p>

<h1 align="center">Freedium: Your paywall breakthrough for Medium!</h1>

<a href="https://www.patreon.com/Freedium">
    <img width="200px" height="50px" alt="Become a Patron" src="https://github.com/elsiehupp/patron-buttons/blob/master/svg/become_a_patron_4x1_black_logo_white_text_on_coral.svg?raw=True">
</a>

## Stack:

- Backend:
  - language: Python 3.12+
  - framework: Uvicorn, Gunicorn, FastAPI
- Frontend:
  - framework: Tailwind CSS v3, Jinja2
  - monitoring: Sentry
- Database:
  - PostgreSQL, Dragonfly (Redis and Memcached compatible key-value database)
- Utils:
  - Caddy, Docker, Docker Compose, Cloudflare WARP proxy (wgcf)

## Project configuration:

There are three (3) docker-compose profiles:

- `min` - Lightweight local setup without Cloudflare WARP proxy clusters, HAProxy balancer, Plausible, or Grafana.
- `local` - Based on `min`, but with `freedium.local` exposed hostname (ports 80 and 443 with self-signed TLS certificate) and reverse proxy.
- `prod` - Full stack including proxies, Plausible analytics, and monitoring for production deployment.

### Requirements:

- Docker & Docker Compose (Docker Compose v2+ recommended)
- Linux (tested on Ubuntu 22.04 & Fedora 39) or Windows/macOS with Docker Desktop
- git

### Local run:

To configure and run your Freedium instance:

1. Clone the repository:

   ```bash
   git clone https://codeberg.org/Freedium-cfd/web/ ./freedium-web --depth 1
   cd ./freedium-web
   ```

2. Create and configure the environment file:

   ```bash
   cp .env_template .env
   ```

   Open `.env` and adjust the configuration as needed:
   - `POSTGRES_PORT`: External PostgreSQL port on the host (default `5432`). Change this (e.g., `5430`) if host port `5432` is already occupied by a local PostgreSQL or another container.
   - `HOST_ADDRESS`: Address of your Freedium instance (default `http://localhost:6752`).
   - `PGADMIN_DEFAULT_EMAIL` / `PGADMIN_DEFAULT_PASSWORD`: Credentials for pgAdmin (accessible at port `5433`).

3. Set up the Docker network:

   ```bash
   docker network create caddy_net
   ```

4. (Optional) Configure domain mapping in your hosts file (for `local` profile):

   ```bash
   # Linux / macOS:
   sudo nano /etc/hosts
   # Windows: edit C:\Windows\System32\drivers\etc\hosts as Administrator
   ```

   Add the following line:

   ```text
   127.0.0.1 freedium.local
   ```

5. Start the Freedium services:

   - **Option A: `local` profile** (with local domain `freedium.local`):
     ```bash
     docker compose --profile local -f ./docker-compose/docker-compose.yml up -d
     ```

   - **Option B: `min` profile** (lightweight setup):
     ```bash
     docker compose --profile min -f ./docker-compose/docker-compose.yml up -d
     ```

   Stopping the services:

   ```bash
   docker compose --profile local -f ./docker-compose/docker-compose.yml down
   # or for min:
   docker compose --profile min -f ./docker-compose/docker-compose.yml down
   ```

### Accessing Services:

- **Freedium Web App**: `https://freedium.local` (or `http://localhost:6752` if accessed directly)
- **Direct Backend**: `http://localhost:7080`
- **pgAdmin**: `http://localhost:5433`
- **PostgreSQL (host tools like DBeaver/psql)**: `localhost:${POSTGRES_PORT}` (default: `5432`, e.g., `5430`)

> [!NOTE]
> If accessing via `https://freedium.local`, your browser may show a warning about an insecure connection because of the self-signed TLS certificate. You can safely proceed.

### Production run:

All production services run on the `prod` profile:

```bash
docker compose --profile prod -f ./docker-compose/docker-compose.yml up -d
```

If you use a separate Dockerized reverse proxy, attach it to the `caddy_freedium_net` network (`external: true`). Route to the `caddy_freedium` hostname on port `6752` (or `6753` for Plausible). See `docker-compose` and `caddy` folders for more details.

## Roadmap

- [ ] Speed up parser logic
- [ ] Parse articles directly to Markdown
- [ ] Add more services than just a Medium
- [ ] Rewrite frontend to Svelte
- [ ] Integrate Grafana/Prometheus/Plausible to monitor our services
- [ ] Translate posts to different languages using `translatepy` library
- [ ] AI-powered summary of posts

[![Star History Chart](https://api.star-history.com/svg?repos=Freedium-cfd/web&type=Date)](https://star-history.com/#Freedium-cfd/web&Date)
