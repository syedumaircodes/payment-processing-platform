# Wallet Service

Source: [`cmd/wallet/main.go`](../../cmd/wallet/main.go)
Introduced in commit `4a83836` — *feat(wallet): add health endpoint with graceful shutdown*

## Purpose

The wallet service is the first application service in the platform. In its current form it is a minimal HTTP service that proves the baseline runtime pattern (logging, routing, graceful shutdown) that subsequent services will follow. No wallet/business logic (balances, transfers, top-ups) exists yet.

## Running

```bash
go run ./cmd/wallet
```

Or build a binary:

```bash
go build -o wallet.exe ./cmd/wallet
```

Listens on **`:8081`**.

## HTTP API

### `GET /healthz`

Health/readiness check.

```
GET /healthz HTTP/1.1
Host: localhost:8081
```

Response:

```
HTTP/1.1 200 OK
Content-Type: application/json

{"status":"ok"}
```

Each request is logged: `msg="health check" remote=<addr>`.

## Behaviour

| Aspect | Implementation |
| --- | --- |
| Logging | `log/slog` text handler → stdout |
| Routing | `http.ServeMux` with method-aware pattern `GET /healthz` (Go 1.22+) |
| Timeouts | `ReadHeaderTimeout: 5s` |
| Startup failure | Logged and `os.Exit(1)` unless the error is `http.ErrServerClosed` |
| Shutdown trigger | `SIGINT` (Ctrl+C) or `SIGTERM` via `signal.NotifyContext` |
| Shutdown window | 10-second deadline for in-flight requests to drain (`srv.Shutdown`) |

## Shutdown Sequence

1. Signal received → context cancelled, log `Shutting down gracefully`.
2. `srv.Shutdown(shutdownCtx)` stops accepting new connections and waits for active handlers (up to 10s).
3. On error: log `Shutdown failed`; otherwise log `Server stopped` and exit 0.

## Configuration

Hard-coded at present: listen address `:8081`. No environment-variable or flag-based configuration has been introduced yet.

## Known Limitations

- No persistence, auth, or business endpoints
- Single-file design — no internal package structure yet
- No tests
