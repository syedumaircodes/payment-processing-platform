# Infrastructure

All local infrastructure is defined in [`docker-compose.yml`](../docker-compose.yml) and runs three services plus one named volume.

## Services

### PostgreSQL 17 (`payments-postgres`)

- **Image:** `postgres:17-alpine`
- **Port:** `5432:5432`
- **Database / user / password:** `payments` / `payments` / `payments` (development credentials — do not reuse outside local dev)
- **Volume:** named volume `pgdata` mounted at `/var/lib/postgresql/data`, so data survives container restarts
- **Healthcheck:** `pg_isready -U payments` every 5s, 3s timeout, 10 retries

### Redis 8 (`payments-redis`)

- **Image:** `redis:8-alpine`
- **Port:** `6379:6379`
- No volume or healthcheck configured (ephemeral local cache is acceptable for development).

### NATS 2.11 (`payments-nats`)

- **Image:** `nats:2.11-alpine`
- **Command:** `-js -m 8222`
  - `-js` enables **JetStream** (durable streams, at-least-once delivery)
  - `-m 8222` exposes the HTTP **monitoring** endpoint
- **Ports:**
  - `4222:4222` — client connections
  - `8222:8222` — monitoring (`/healthz`, `/varz`, etc.)

### Volumes

| Volume | Purpose |
| --- | --- |
| `pgdata` | Persists PostgreSQL data across container recreation |

## Usage

```bash
# Start all services in the background
docker compose up -d

# Check status and health
docker compose ps

# Follow logs
docker compose logs -f

# Stop (data in pgdata is retained)
docker compose down

# Stop and DELETE persisted Postgres data
docker compose down -v
```

## Current Integration Status

| Service | Used by application code? |
| --- | --- |
| PostgreSQL | No — no driver or connection code yet |
| Redis | No |
| NATS | No |

The wallet service currently runs standalone and does not depend on any of these containers.
