# Binance AI Data Collector

## Goal

Build a lightweight, production-ready platform for collecting Binance market data.

The system is responsible for:

- Collecting historical and real-time market data.
- Validating and normalizing incoming data.
- Persisting immutable market data.
- Providing data for future AI training.
- Sending system notifications.
- Exposing REST APIs for monitoring.

Out of Scope

- Trading
- Prediction
- Technical Indicators
- Strategy
- Feature Engineering

---

# Technology

- Python 3.13+
- FastAPI
- SQLAlchemy
- PostgreSQL / TimescaleDB
- Docker Compose
- Binance REST API
- Binance WebSocket API
- Telegram Bot

---

# Environment and Setup

## Required configuration

The implementation should support the following configuration inputs:
- Binance REST and WebSocket endpoints
- API and Telegram settings
- Database connection settings
- Logging and retry configuration
- Symbol and interval configuration

Configuration must be loaded from .env and never hardcoded.

## Development workflow

- Implement one phase at a time.
- Keep changes scoped to the current milestone.
- Reuse existing modules before introducing new abstractions.
- Preserve the documented architecture and data rules.

## Testing expectations

- Unit tests should cover validators, normalizers, and repository behavior.
- Integration tests should validate collector-to-persistence flow.
- API and Telegram components should be tested for correct status and notification behavior.

## Definition of done

A phase is considered complete when:
- the required functionality is implemented,
- the change follows the documented architecture,
- data integrity and immutability rules are preserved,
- relevant documentation is updated if behavior changes.

---

# Data Sources

REST

- ExchangeInfo
- Klines
- Depth
- Trades
- BookTicker
- 24hr Ticker

WebSocket

- trade
- aggTrade
- depth
- bookTicker
- kline
- miniTicker
- ticker

Rules

- Prefer WebSocket.
- Use REST for bootstrap, historical data and recovery.

---

# Data Types

Store the following datasets.

- Symbols
- Ticker
- BookTicker
- OrderBook
- Trades
- Aggregate Trades
- Klines

Future Extension

- Funding Rate
- Open Interest
- Liquidation
- Mark Price

---

# Database

Time-series database.
Market data is immutable.
Only INSERT operations.
Never UPDATE.
Never DELETE.
Use batch writing.
Deduplicate before persistence.
Index
(symbol, event_time)

---

# Reliability

- Auto reconnect.
- Auto retry.
- Heartbeat monitoring.
- Automatic gap detection.
- Automatic historical recovery.
- Graceful shutdown.

---

# Telegram

Commands

- /status
- /reload
- /history
- /config

Notifications

- Startup
- Shutdown
- Connection Lost
- Recovery Started
- Recovery Finished
- Database Error

---

# REST API
GET /status
GET /history
GET /config