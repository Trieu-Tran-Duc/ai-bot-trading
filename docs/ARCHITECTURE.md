# Architecture

## Overview

Binance AI Data Collector is a lightweight, event-driven data platform.

The system collects historical and real-time market data from Binance, validates and normalizes it, then persists immutable data for future AI training.

The system is designed to be modular, asynchronous, and easy to extend.

---

# High-Level Architecture

```
                    Binance

          REST API + WebSocket
                    │
                    ▼
              Data Collector
                    │
                    ▼
               Data Pipeline
                    │
     ┌──────────────┴──────────────┐
     ▼                             ▼
 Database Writer              Event Bus
     │                             │
     ▼                             ▼
 PostgreSQL                  Telegram
     │
     ▼
 FastAPI
```
---

# System Components

## Collector

Responsibilities

- Connect to Binance REST API.
- Connect to Binance WebSocket.
- Receive market data.
- Handle reconnect and recovery.
- Push raw events into the pipeline.

Collector must not:

- Store data directly.
- Calculate indicators.
- Generate predictions.
- Execute business logic.

---

## Pipeline

Responsibilities

- Validate incoming messages.
- Normalize Binance payloads.
- Convert data into internal models.
- Forward valid records to the database writer.

Pipeline must remain stateless.

---

## Database Writer

Responsibilities

- Receive normalized records.
- Deduplicate data.
- Persist data.
- Batch writes when appropriate.

Database Writer is the only component allowed to write to the database.

---

## Database

Responsibilities

- Store immutable market data.
- Store metadata.
- Store system logs.

Market data must never be modified after insertion.

---

## Telegram

Responsibilities

- Receive system events.
- Send notifications.
- Execute administrative commands.

Telegram must never access the database directly.

---

## REST API

Responsibilities

- Provide health status.
- Provide historical data.
- Provide configuration information.

REST API is read-only.

REST API must never communicate directly with Binance.

---

# Data Flow

Realtime

```
Binance WebSocket
        │
Receive Event
        │
Validate
        │
Normalize
        │
Buffer
        │
Persist
        │
Notify
```

Historical

```
REST Request
      │
Download
      │
Validate
      │
Normalize
      │
Persist
```

---

# Dependency Rules

```
Collector
        │
        ▼
Pipeline
        │
        ▼
Database Writer
        │
        ▼
Repository
        │
        ▼
Database
```

Allowed dependencies
Collector
→ Pipeline
Pipeline
→ Database Writer
Database Writer
→ Repository
Repository
→ Database
Forbidden
REST API
✗ Database Models
REST API
✗ Binance
Telegram
✗ Database
Collector
✗ Repository
Collector
✗ Database

---

# Design Principles

- Single Responsibility Principle
- Separation of Concerns
- Dependency Injection
- Async First
- Event Driven
- Immutable Market Data
- Configuration Driven
- Production Ready

---

# Scalability

The architecture must support:

- Multiple symbols
- Multiple intervals
- Multiple WebSocket streams
- Multiple REST collectors
- Additional exchanges
- Additional notification channels

No architecture changes should be required to support these extensions.

---

# Extension Points

The following modules are designed for extension:

Collector

- Binance Spot
- Binance Futures
- Other Exchanges

Notifier

- Telegram
- Discord
- Slack

Storage

- PostgreSQL
- TimescaleDB

API

- REST
- WebSocket API

Each extension must implement the existing interface without modifying the core architecture.

---

# Non-Goals

The architecture does not include:

- Trading Engine
- AI Engine
- Machine Learning
- Technical Indicators
- Strategy Engine
- Risk Management

These components will be implemented as independent services in future phases.

---

# Architecture Constraints

- Market data is immutable.
- Collector never writes directly to the database.
- REST API never accesses Binance.
- Database Writer is the only persistence layer.
- All communication is asynchronous whenever possible.
- Business logic must remain outside infrastructure components.
- Every component must have a single responsibility.