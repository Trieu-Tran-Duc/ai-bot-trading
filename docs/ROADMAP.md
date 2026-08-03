# Roadmap

## Development Rules

- Complete one phase at a time.
- Do not implement future phases early.
- Every phase must be reviewed before starting the next.
- Keep each commit focused on a single phase.
- All code must follow docs/PROJECT_RULES.md.

## Implementation Expectations

- Complete one phase at a time.
- Do not implement future phases early.
- Keep each change scoped to the current phase.
- Preserve the documented architecture, data rules, and configuration conventions.
- Update the relevant documentation when behavior or interfaces change.

## Overview

This roadmap defines the implementation order of the project.

Complete each phase before moving to the next.

Do not skip phases.

## Review Gate for Every Phase

Before moving to the next phase, each phase must complete the following:

- Review
- Refactor
- Unit Test
- Documentation Update
- Approval

A phase is not considered complete until all five items are satisfied.

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

## Review

- Architecture and boundaries reviewed
- Configuration and logging reviewed

## Refactor

- Code simplified where possible
- Duplication removed

## Unit Test

- Configuration and startup tests added
- Basic dependency smoke tests covered

## Documentation Update

- Project docs updated to reflect the implementation

## Approval

- Phase review approved

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

## Review

- Schema and repository boundaries reviewed
- Data immutability reviewed

## Refactor

- Repository interfaces simplified
- Batch write logic refined

## Unit Test

- Repository insert and deduplication tests added
- Connection and migration smoke tests added

## Documentation Update

- Database and persistence documentation updated

## Approval

- Phase review approved

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

## Review

- Endpoint contract and response handling reviewed
- Rate limit behavior reviewed

## Refactor

- Retry handling cleaned up
- Client abstractions simplified

## Unit Test

- REST client error and retry tests added
- Response validation tests added

## Documentation Update

- Binance data and requirements docs updated

## Approval

- Phase review approved

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

## Review

- Stream lifecycle and reconnection reviewed
- Backpressure and buffering reviewed

## Refactor

- Reconnect flow simplified
- Stream handler responsibilities clarified

## Unit Test

- Reconnect and resubscribe tests added
- Heartbeat handling tests added

## Documentation Update

- Architecture and Binance data docs updated

## Approval

- Phase review approved

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

## Review

- Validation and normalization rules reviewed
- Buffering and deduplication reviewed

## Refactor

- Pipeline stages simplified
- Error propagation refined

## Unit Test

- Validator and normalizer unit tests added
- Duplicate detection tests added

## Documentation Update

- Pipeline and data rules docs updated

## Approval

- Phase review approved

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

## Review

- Write path and transaction boundaries reviewed
- Idempotency reviewed

## Refactor

- Batch writer simplified
- Conflict handling improved

## Unit Test

- Batch write and conflict tests added
- Repository-level deduplication tests added

## Documentation Update

- Database writer and database docs updated

## Approval

- Phase review approved

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

## Review

- Job scheduling and recovery behavior reviewed
- Failure handling reviewed

## Refactor

- Scheduler responsibilities clarified
- Recovery flow simplified

## Unit Test

- Job execution and retry tests added
- Recovery path tests added

## Documentation Update

- Requirements and roadmap docs updated

## Approval

- Phase review approved

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

## Review

- Notification and command handling reviewed
- Error reporting reviewed

## Refactor

- Telegram handlers simplified
- Notification payloads standardized

## Unit Test

- Notification and command tests added
- Error-reporting tests added

## Documentation Update

- Requirements and architecture docs updated

## Approval

- Phase review approved

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

## Review

- Endpoint contracts reviewed
- Read-only security reviewed

## Refactor

- Endpoint organization simplified
- Response models standardized

## Unit Test

- API route tests added
- Health and status tests added

## Documentation Update

- API requirements and architecture docs updated

## Approval

- Phase review approved

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

## Review

- Health checks and monitoring reviewed
- Alerting path reviewed

## Refactor

- Monitoring endpoints simplified
- Status aggregation refined

## Unit Test

- Health and monitoring tests added
- Status aggregation tests added

## Documentation Update

- Monitoring and architecture docs updated

## Approval

- Phase review approved

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

## Review

- Test coverage reviewed
- Regression risks reviewed

## Refactor

- Test helpers simplified
- Flaky tests stabilized

## Unit Test

- Unit and integration tests completed
- Regression suite added

## Documentation Update

- Testing and implementation docs updated

## Approval

- Phase review approved

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

## Review

- Deployment and operations reviewed
- Backup and recovery reviewed

## Refactor

- Deployment configuration refined
- Operational scripts simplified

## Unit Test

- Deployment smoke tests added
- Operational recovery tests added

## Documentation Update

- Production and operations docs updated

## Approval

- Phase review approved

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