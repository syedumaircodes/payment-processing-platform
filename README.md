# Payments Processing Platform

A payments backend in Go. This project is early stage — see [Documentation](docs/) for details.

## Current state

- **Go module** — `github.com/syedumaircodes/payments-processing-platform` (Go 1.27.3)
- **Local infrastructure** via Docker Compose:
  - PostgreSQL 17 (with persistent volume and healthcheck)
  - Redis 8
  - NATS 2.11 with JetStream enabled
- **Wallet service** — `cmd/wallet`, an HTTP service on port `:8081`

## Project layout

```
payment-processing-platform/
├── cmd/wallet/main.go    # Wallet service entry point
├── docker-compose.yml    # Local development infrastructure
├── go.mod
└── docs/                 # Documentation
```

## Getting started

### Prerequisites

- Go 1.27+
- Docker + Docker Compose

### Run

```bash
# Start infrastructure (Postgres, Redis, NATS)
docker compose up -d

# Run the wallet service
go run ./cmd/wallet

# Health check
curl http://localhost:8081/healthz
# {"status":"ok"}
```

### Stop

The wallet service shuts down gracefully on `Ctrl+C`. For infrastructure:

```bash
docker compose down      # keep data
docker compose down -v   # delete Postgres data
```

## Documentation

- [Architecture](docs/architecture.md)
- [Infrastructure](docs/infrastructure.md)
- [Wallet service](docs/services/wallet.md)
- [Development guide](docs/development.md)
