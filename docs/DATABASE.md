# Database Specification

## Purpose

This document defines the database schema used by the Binance AI Data Collector.

The database stores immutable market data for future AI training and analysis.

Market data is append-only.

Raw market data must never be modified after insertion.

---

# Database Engine

Primary Database
- PostgreSQL

Recommended
- TimescaleDB

Timezone
- UTC

Character Set
- UTF-8

---

# General Rules

- Store raw market data.
- Store normalized fields.
- Never UPDATE market data.
- Never DELETE market data.
- INSERT only.
- Deduplicate before insert.
- Use transactions for batch writes.
- Every table has `created_at`.
- Every timestamp uses UTC.

---

# Naming Convention

Table
snake_case
Column
snake_case
Primary Key
id
Foreign Key
<table>_id
Timestamp
event_time

---

# Tables

## symbols

Purpose
Store Binance trading symbol metadata.
Primary Key
id
Unique
symbol
Fields

- symbol
- base_asset
- quote_asset
- status
- is_spot_trading_allowed
- is_margin_trading_allowed
- raw_data
- created_at

Indexes
- symbol

Refresh
Startup

---

## ticker

Purpose
Store real-time ticker information.
Primary Key
id
Unique
(symbol, event_time)

Fields

- symbol
- last_price
- price_change
- price_change_percent
- weighted_avg_price
- volume
- quote_volume
- event_time
- raw_data
- created_at

Indexes

- symbol
- event_time
- (symbol, event_time)

Source
REST
WebSocket

---

## book_ticker

Purpose
Store best bid and ask.
Primary Key
id
Unique
(symbol, event_time)
Fields

- symbol
- bid_price
- bid_quantity
- ask_price
- ask_quantity
- event_time
- raw_data
- created_at

Indexes

- symbol
- event_time

Source
REST
WebSocket

---

## orderbook

Purpose
Store order book snapshots and updates.
Primary Key
id
Fields

- symbol
- last_update_id
- bids
- asks
- event_time
- raw_data
- created_at

Indexes

- symbol
- event_time

Source
REST
WebSocket

---

## trades

Purpose
Store market trades.
Primary Key
id
Unique
trade_id
Fields

- trade_id
- symbol
- price
- quantity
- buyer_maker
- event_time
- raw_data
- created_at

Indexes

- trade_id
- symbol
- event_time

Source
REST
WebSocket

---

## agg_trades

Purpose
Store aggregate trades.
Primary Key
id
Unique
aggregate_trade_id
Fields

- aggregate_trade_id
- symbol
- price
- quantity
- first_trade_id
- last_trade_id
- event_time
- raw_data
- created_at

Indexes

- symbol
- event_time

Source
REST
WebSocket

---

## klines

Purpose
Store candlestick data.
Primary Key
id
Unique
(symbol, interval, open_time)

Fields

- symbol
- interval
- open_time
- close_time
- open
- high
- low
- close
- volume
- quote_volume
- trade_count
- taker_buy_base_volume
- taker_buy_quote_volume
- event_time
- raw_data
- created_at

Indexes

- symbol
- interval
- open_time

Source
REST
WebSocket

---

## collector_logs

Purpose
Store collector events.
Primary Key
id
Fields

- level
- module
- message
- exception
- created_at

Indexes

- level
- created_at

---

# Standard Columns

Every market table must contain

- symbol
- event_time
- raw_data
- created_at

---

# Data Types

Price
Decimal
Quantity
Decimal
Volume
Decimal
Timestamp
BIGINT (milliseconds)
Boolean
BOOLEAN
Raw Data
JSONB

---

# Batch Writing

Database writes must support batching.
The persistence layer is responsible for:

- buffering
- deduplication
- transaction management

---

# Constraints

Market data is immutable.
No cascading delete.
No business logic inside database models.
Database models only represent stored data.

---

# Repository and Write Contract

The repository layer is the only layer that should access persistence directly.

Responsibilities:
- execute inserts and batch writes
- enforce deduplication rules
- manage transactions and write buffering
- expose read operations for services and APIs

The collector and pipeline layers must not write directly to the database.

# Migration and Schema Evolution

- Schema changes should be introduced through explicit migration steps.
- Existing market data should remain append-only and backward-compatible.
- New columns or tables should not break existing consumers without a documented migration plan.

# Operational Notes

- Use UTC timestamps consistently.
- Keep indexes aligned with query patterns such as symbol and event_time.
- Prefer bulk inserts and batched persistence for throughput.
- Preserve raw payloads and metadata for auditability.

---

# Future Tables

Reserved for future implementation.

- funding_rates
- open_interest
- mark_price
- liquidation
- predictions
- ai_models
- ai_predictions

---

# Performance Guidelines

- Use indexes on frequently queried columns.
- Optimize for INSERT performance.
- Avoid unnecessary joins.
- Prefer time-based queries.
- Archive historical data when required.

---

# Database Ownership

Collector
Read ❌
Write ❌
Repository
Read ✅
Write ✅
Service
Read ✅
Write through Repository only
API
Read through Service only
Telegram
No database access
AI
Read only