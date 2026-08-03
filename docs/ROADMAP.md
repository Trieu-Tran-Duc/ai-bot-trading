# Roadmap

## Development Rules

- Complete one phase at a time.
- Do not implement future phases early.
- Every phase must be reviewed before starting the next.
- Keep each commit focused on a single phase.
- All code must follow docs/PROJECT_RULES.md.

## Overview

This roadmap defines the implementation order of the project.

Complete each phase before moving to the next.

Do not skip phases.

---

# Phase 1 — Project Foundation

## Goal

Initialize the project structure and development environment.

## Tasks

- Create project structure
- Configure Python project
- Configure Docker Compose
- Configure environment variables
- Configure logging
- Configure dependency injection

## Done

- Project builds successfully
- Docker starts successfully
- Configuration loads correctly

---

# Phase 2 — Database

## Goal

Prepare the persistence layer.

## Tasks

- Configure PostgreSQL
- Configure SQLAlchemy
- Create database session
- Create base model
- Create repositories
- Configure migrations

## Done

- Database connection successful
- Migrations run successfully
- Repository layer completed

---

# Phase 3 — Binance REST Client

## Goal

Implement Binance REST integration.

## Tasks

- ExchangeInfo
- Klines
- Trades
- OrderBook Snapshot
- BookTicker

## Done

- Async REST client
- Retry
- Timeout
- Error handling
- Response validation

---

# Phase 4 — Binance WebSocket Client

## Goal

Implement real-time data collection.

## Tasks

- Trade stream
- AggTrade stream
- BookTicker stream
- Depth stream
- Kline stream
- Combined streams

## Done

- Auto reconnect
- Auto resubscribe
- Heartbeat monitoring
- Graceful shutdown

---

# Phase 5 — Data Pipeline

## Goal

Process incoming market data.

## Tasks

- Validator
- Normalizer
- Deduplicator
- Buffer

## Done

- Invalid messages rejected
- Data normalized
- Buffer operational

---

# Phase 6 — Database Writer

## Goal

Persist normalized data.

## Tasks

- Batch writer
- Transaction management
- Bulk insert
- Conflict handling

## Done

- Batch insert works
- No duplicate records
- High-throughput writing

---

# Phase 7 — Scheduler

## Goal

Execute background jobs.

## Tasks

- Historical collection
- Snapshot updates
- Health checks
- Recovery jobs

## Done

- Scheduler running
- Jobs execute correctly

---

# Phase 8 — Telegram

## Goal

Implement monitoring and notifications.

## Tasks

- Notifications
- Commands
- Error reporting
- Status reporting

## Done

- Commands operational
- Notifications delivered

---

# Phase 9 — REST API

## Goal

Expose monitoring endpoints.

## Endpoints

- GET /status
- GET /history
- GET /config

## Done

- API documented
- API tested
- Read-only access

---

# Phase 10 — Monitoring

## Goal

Monitor system health.

## Tasks

- Health check
- Collector status
- Database status
- WebSocket status
- REST status

## Done

- Health endpoint available
- Monitoring complete

---

# Phase 11 — Testing

## Goal

Verify system stability.

## Tasks

- Unit tests
- Integration tests
- Collector tests
- Repository tests
- API tests

## Done

- All tests pass
- No critical issues

---

# Phase 12 — Production

## Goal

Prepare production deployment.

## Tasks

- Docker optimization
- Environment validation
- Logging verification
- Backup strategy
- Performance review

## Done

- Production ready
- Documentation complete

---

# Future Roadmap

These features are intentionally excluded from the current project.

## AI Dataset

- Feature engineering
- Dataset generation
- Data labeling

## AI Engine

- Model training
- Model evaluation
- Model versioning

## Prediction Service

- Price prediction
- Confidence score
- Signal generation

## Trading Engine

- Order execution
- Risk management
- Portfolio management

## Dashboard

- Web UI
- Charts
- Metrics
- Live monitoring