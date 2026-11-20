# Development Guide

## Prerequisites

- **Go 1.27.3+** (per `go.mod`)
- **Docker + Docker Compose** (for Postgres, Redis, NATS)
- Git

## Project Layout

```
payment-processing-platform/
├── cmd/
│   └── wallet/          # Entry points for services (one per service)
│       └── main.go
├── docker-compose.yml   # Local infrastructure
├── go.mod               # Module: github.com/syedumaircodes/payments-processing-platform
└── docs/                # Documentation
```

Conventions so far:

- One binary per service under `cmd/<service>/main.go`
- Conventional commits: `feat(wallet): …`, `chore: …`

## Getting Started

```bash
# 1. Clone and enter the repository
git clone <repo-url>
cd payment-processing-platform

# 2. Start infrastructure (optional for the wallet service — it has no dependencies yet)
docker compose up -d

# 3. Run the wallet service
go run ./cmd/wallet

# 4. Verify
curl http://localhost:8081/healthz
# {"status":"ok"}
```

## Building

```bash
# Single service
go build -o wallet.exe ./cmd/wallet

# Everything
go build ./...
```

## Testing

```bash
go test ./...
```

> Note: no test files exist as of the last commit.

## Linting / Formatting

```bash
gofmt -l .
go vet ./...
```

## Stopping

The wallet service handles `Ctrl+C` gracefully (drains requests for up to 10 seconds). Infrastructure:

```bash
docker compose down      # keep data
docker compose down -v   # delete Postgres data
```

## Adding a New Service

1. Create `cmd/<name>/main.go`.
2. Follow the wallet service pattern: `slog` logger, `http.ServeMux` with method-aware routes, `signal.NotifyContext` + `srv.Shutdown` with a 10s timeout, and `ReadHeaderTimeout`.
3. Choose a port not already in use (wallet uses `8081`).
4. Document it under `docs/services/`.
