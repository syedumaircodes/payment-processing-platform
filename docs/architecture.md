# Architecture

## Overview

The payments processing platform is being built as a Go monorepo containing multiple services, backed by shared infrastructure run via Docker Compose. As of the last commit (`4a83836`), the repository contains the infrastructure definition and a single service skeleton (the wallet service).

## Current State

```
payment-processing-platform/
├── docker-compose.yml      # Local infrastructure (Postgres, Redis, NATS)
├── go.mod                  # Go module definition
├── cmd/
│   └── wallet/
│       └── main.go         # Wallet HTTP service (health endpoint + graceful shutdown)
└── docs/                   # This documentation
```

## Planned Infrastructure Roles

Based on the services defined in `docker-compose.yml`, the intended architecture is:

| Component             | Version              | Intended Role                                                                                  |
| --------------------- | -------------------- | ---------------------------------------------------------------------------------------------- |
| PostgreSQL 17         | `postgres:17-alpine` | System of record: accounts, transactions, ledger entries                                       |
| Redis 8               | `redis:8-alpine`     | Low-latency caching, rate limiting, idempotency keys, short-lived locks                        |
| NATS 2.11 (JetStream) | `nats:2.11-alpine`   | Async messaging: event-driven workflows (payment initiated/settled/failed), service decoupling |

NATS is started with `-js` (JetStream enabled) and `-m 8222` (monitoring endpoint), indicating durable, at-least-once event delivery is planned rather than fire-and-forget pub/sub.

## Current System Diagram

```
                 ┌─────────────────────────────┐
                 │        docker compose       │
                 │                             │
  client ──────► │  wallet service  :8081      │
   (GET          │    └─ GET /healthz          │
    /healthz)    │                             │
                 │  postgres :5432  (pgdata)   │  (defined, unused by code yet)
                 │  redis    :6379            │  (defined, unused by code yet)
                 │  nats     :4222 / 8222     │  (defined, unused by code yet)
                 └─────────────────────────────┘
```

Only the wallet service has application code. The data services are provisioned and healthy but are not yet connected to any code.

## Wallet Service Design

The wallet service (`cmd/wallet/main.go`) is a single-file Go program demonstrating the baseline pattern expected of future services:

1. **Structured logging** via `log/slog` with a text handler writing to stdout.
2. **HTTP server** using `net/http` with method+path routing (`GET /healthz`) and a 5-second `ReadHeaderTimeout`.
3. **Graceful shutdown** driven by `signal.NotifyContext` listening for `SIGINT`/`SIGTERM`, followed by `srv.Shutdown` with a 10-second deadline.
4. **Fail-fast startup errors** — a non-`ErrServerClosed` listen error is logged and exits with status 1.
