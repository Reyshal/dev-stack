# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A local development infrastructure stack defined entirely in `docker-compose.yml` — there is no application code, build, lint, or test suite. Work here means editing the compose file and keeping `README.md` in sync with it.

Each data service is paired with a browser-based viewer, all on the shared `dev-stack-network` bridge network:

| Service | Host port | Viewer | Viewer port |
|---|---|---|---|
| MySQL 8.0 | 3306 | phpMyAdmin | 8090 |
| Elasticsearch 8.11.0 | 9200 / 9300 | Elasticvue | 8092 |
| Memcached 1.6 | 11211 | phpCacheAdmin | 8091 |
| Redis 7 | 6379 | RedisInsight | 5540 |

## Commands

```bash
docker compose config -q                  # validate the compose file after editing
docker compose up -d                      # start everything
docker compose up -d <service> [<viewer>] # (re)create only what changed
docker compose ps
docker compose logs -f <service>
curl http://localhost:9200/_cluster/health
```

## Things that span multiple files / aren't obvious

- **Adding a service:** follow the existing pattern — `container_name: dev-stack-<name>`, `restart: unless-stopped`, `networks: [dev-stack-network]`, a named volume declared under top-level `volumes:` if it persists data, and `depends_on` for viewers. Then update every relevant README section: Features list, Services section, Connection Details / "Web Management Interfaces", and the Troubleshooting alternate-port list (keep alternate ports from colliding with real ones).
- **Elasticvue ↔ Elasticsearch coupling:** Elasticvue runs in the user's browser and calls `http://localhost:9200` directly, so Elasticsearch has `http.cors.*` env vars whose `allow-origin` must match Elasticvue's host port (8092). Change one, change the other.
- **MySQL init scripts:** `./mysql-init` is mounted to `/docker-entrypoint-initdb.d` and its `.sql` files run in alphabetical order **only when the `mysql_data` volume is empty**. Re-importing requires `docker compose down -v` (destroys all service volumes) or removing just the MySQL volume. The local `mysql-init/*.sql` dumps are gitignored and very large (multiple GB) — never read them whole or try to commit them.
- **User-creation template:** the tracked example lives at `mysql-init-example/99_init_user.sql.example` (moved in a refactor). The README and `.gitignore` still reference the old path `mysql-init/99_init-user.sql.example`; fix those if touching that area.
- Credentials (`root`/`rootpassword`, Elasticsearch security disabled) are intentional dev-only defaults.
