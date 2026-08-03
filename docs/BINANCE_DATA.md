# Binance Data Specification

## Purpose

Define all Binance market data used by the project.
This document is the single source of truth for:

- Binance endpoints
- WebSocket streams
- Database mapping
- Collection strategy

Only public market data is used.
No authenticated endpoints.

---

# API Base URL

REST
https://api.binance.com
WebSocket
wss://stream.binance.com:9443
Reference
https://developers.binance.com/en/docs/products/spot/rest-api
https://developers.binance.com/en/docs/binance-spot-api-docs/web-socket-streams

---

# Collection Strategy

| Data | Source | Method |
|--------|---------|---------|
| Exchange Info | REST | Startup |
| Historical Klines | REST | On Demand |
| Historical Trades | REST | Recovery |
| OrderBook Snapshot | REST | Recovery |
| Market Stream | WebSocket | Realtime |

Priority

1. WebSocket
2. REST Recovery
3. REST Historical

---

# REST Endpoints

| Endpoint | Purpose | Store |
|----------|----------|--------|
| GET /api/v3/exchangeInfo | Symbol metadata | symbols |
| GET /api/v3/klines | Historical candles | klines |
| GET /api/v3/depth | OrderBook snapshot | orderbook |
| GET /api/v3/trades | Recent trades | trades |
| GET /api/v3/historicalTrades | Recovery | trades |
| GET /api/v3/aggTrades | Aggregate trades | agg_trades |
| GET /api/v3/ticker/24hr | Market statistics | ticker |
| GET /api/v3/ticker/bookTicker | Best Bid/Ask | book_ticker |
| GET /api/v3/ticker/price | Last Price | ticker |

---

# WebSocket Streams

| Stream | Purpose | Store |
|---------|----------|--------|
| trade | Real-time trades | trades |
| aggTrade | Aggregate trades | agg_trades |
| bookTicker | Best Bid / Ask | book_ticker |
| depth | OrderBook updates | orderbook |
| kline | Candlestick | klines |
| miniTicker | Price summary | ticker |
| ticker | 24hr statistics | ticker |

---

# Supported Symbols

Loaded dynamically from
GET /api/v3/exchangeInfo
Never hardcode symbols.

---

# Supported Intervals

5m
15m
30m
1h
Loaded from configuration.

---

# Database Mapping

| Binance | Database |
|----------|----------|
| exchangeInfo | symbols |
| trade | trades |
| aggTrade | agg_trades |
| depth | orderbook |
| bookTicker | book_ticker |
| kline | klines |
| miniTicker | ticker |
| ticker | ticker |

---

# Standard Fields

Every record should contain
event_time
symbol
source
received_at
created_at
raw_data

---

# Data Rules

Store UTC timestamps.
Store original Binance event time.
Store raw response.
Store normalized data.
Never overwrite market data.
Never delete market data.
Deduplicate before insert.

---

# WebSocket Rules

Use combined streams whenever possible.
Automatically reconnect.
Automatically resubscribe.
Monitor heartbeat.
Detect missing events.
Recover missing data using REST.

---

# REST Rules

Use only for

- Startup
- Historical data
- Snapshot
- Recovery

Respect Binance rate limits.
Retry transient failures.
Never poll continuously for realtime data.

---

# Data Quality

Reject

- Invalid symbol
- Invalid timestamp
- Empty payload
- Duplicate event

Normalize

- Timestamp
- Decimal
- Symbol
- Event type

---

# Normalization Contract

Every incoming Binance event should be normalized into a consistent internal record before persistence.

Required fields:
- event_time
- symbol
- source
- received_at
- created_at
- raw_data

Normalization rules:
- Convert timestamps to UTC.
- Preserve the original Binance event time as metadata when available.
- Normalize symbol casing and value types where needed.
- Retain the raw payload for traceability and replay.

# Deduplication and Idempotency

The collector and writer layers must prevent duplicate persistence.

Rules:
- Deduplicate before insert.
- Use stable event identifiers where available.
- Avoid overwriting existing raw market data.
- Treat retries as idempotent operations.

# Operational Requirements

- Respect Binance rate limits and retry with backoff for transient errors.
- Automatically reconnect and resubscribe for WebSocket streams.
- Detect missing events and recover using REST when appropriate.
- Keep all processing asynchronous and non-blocking where possible.

---

# Future Extensions

Spot
✅
Margin
⬜
USD-M Futures
⬜
COIN-M Futures
⬜
Options
⬜
---

# Official References

Spot REST API
https://developers.binance.com/en/docs/products/spot/rest-api
Spot WebSocket Streams
https://developers.binance.com/en/docs/binance-spot-api-docs/web-socket-streams
Spot API Reference
https://developers.binance.com/en/docs/products/binance-open-api/apis
Spot API Swagger
https://binance.github.io/binance-api-swagger/