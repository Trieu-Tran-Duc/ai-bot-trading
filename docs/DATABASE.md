# DATABASE

PostgreSQL = **SSOT**. Chi tiết integration → INTEGRATIONS.md.

## Principles (tóm tắt)

- UTC timestamps · Validate+normalize trước persist · Raw immutable · Dedupe idempotent
- Chỉ Repository layer được gọi DB · Migrations qua Alembic

## Tables (Phase 1)

| Table | Purpose | Unique key |
|-------|---------|------------|
| `symbols` | Trading pairs | `symbol` |
| `intervals` | Candle intervals | `name` |
| `klines` | OHLCV | `(symbol_id, interval_id, open_time)` |
| `agg_trades` | Aggregated trades | `(symbol_id, agg_trade_id)` |
| `collector_status` | Runtime state | `collector_name` |
| `sync_history` | Backfill log | — |
| `audit_logs` | System events | — |

## DDL — Core

```sql
CREATE TABLE symbols (
    id SERIAL PRIMARY KEY,
    symbol VARCHAR(20) NOT NULL UNIQUE,
    base_asset VARCHAR(10) NOT NULL,
    quote_asset VARCHAR(10) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE intervals (
    id SERIAL PRIMARY KEY,
    name VARCHAR(5) NOT NULL UNIQUE  -- 1m, 5m, 1h
);

CREATE TABLE klines (
    id BIGSERIAL,
    symbol_id INT NOT NULL REFERENCES symbols(id),
    interval_id INT NOT NULL REFERENCES intervals(id),
    open_time TIMESTAMPTZ NOT NULL,
    open NUMERIC(20,8) NOT NULL,
    high NUMERIC(20,8) NOT NULL,
    low NUMERIC(20,8) NOT NULL,
    close NUMERIC(20,8) NOT NULL,
    volume NUMERIC(30,8) NOT NULL,
    quote_volume NUMERIC(30,8) NOT NULL,
    trade_count INT DEFAULT 0,
    close_time TIMESTAMPTZ NOT NULL,
    ingested_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (symbol_id, interval_id, open_time)
);

CREATE INDEX idx_klines_lookup ON klines (symbol_id, interval_id, open_time DESC);

CREATE TABLE agg_trades (
    id BIGSERIAL,
    symbol_id INT NOT NULL REFERENCES symbols(id),
    agg_trade_id BIGINT NOT NULL,
    price NUMERIC(20,8) NOT NULL,
    quantity NUMERIC(30,8) NOT NULL,
    trade_time TIMESTAMPTZ NOT NULL,
    is_buyer_maker BOOLEAN NOT NULL,
    ingested_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (symbol_id, agg_trade_id)
);
```

Seed: BTCUSDT, ETHUSDT, SOLUSDT, BNBUSDT + interval `1m`.

## Insert rules

```sql
INSERT INTO klines (...) VALUES (...)
ON CONFLICT (symbol_id, interval_id, open_time) DO NOTHING;
```

- Kline: chỉ closed candle
- Batch writes trong transaction
- Gap: detect → backfill REST, không invent data

## View — AI training

```sql
CREATE VIEW v_klines_training AS
SELECT s.symbol, i.name AS interval, k.open_time,
       k.open, k.high, k.low, k.close, k.volume, k.quote_volume, k.close_time
FROM klines k
JOIN symbols s ON s.id = k.symbol_id
JOIN intervals i ON i.id = k.interval_id
WHERE s.is_active;
```

## Naming

Tables/columns: `snake_case`, plural tables · Index: `idx_{table}_{col}` · FK: `fk_{src}_{tgt}`

## ORM mapping

| Table | Model | Path |
|-------|-------|------|
| symbols | `Symbol` | `app/database/models/symbol.py` |
| klines | `Kline` | `app/database/models/kline.py` |
| agg_trades | `AggTrade` | `app/database/models/agg_trade.py` |

Schema change → migration + cập nhật file này.

## Phase 2 tables

Predictions, model registry, evaluations → [PHASE2-DATABASE.md](PHASE2-DATABASE.md)
