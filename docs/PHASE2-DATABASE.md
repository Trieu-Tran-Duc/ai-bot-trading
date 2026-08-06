# Phase 2 — DATABASE

AI tables bổ sung Phase 1. Market data read-only → [DATABASE.md](DATABASE.md).

## Principles

- Incremental processing via `ai_pipeline_state` watermark
- Sheets sync cursor via `sheets_sync_state`
- Predictions immutable; eval append/update eval row only
- Artifacts on disk (`models/`), metadata in `ml_models`

## Tables

| Table | Purpose |
|-------|---------|
| `ai_pipeline_state` | Watermark + last job status per (symbol, interval) |
| `ml_models` | Registry, metrics, is_active |
| `ml_training_runs` | Train job log |
| `ml_benchmark_results` | Per-model metrics |
| `predictions` | Forecast output |
| `prediction_evaluations` | Actual vs predicted |
| `sheets_sync_state` | Last synced prediction id / timestamp |
| `feature_cache` | Optional incremental feature rows |

## DDL — Pipeline state

```sql
CREATE TABLE ai_pipeline_state (
    symbol_id INT NOT NULL REFERENCES symbols(id),
    interval_id INT NOT NULL REFERENCES intervals(id),
    last_processed_open_time TIMESTAMPTZ NOT NULL,
    last_kline_close_time TIMESTAMPTZ,
    last_feature_refresh_at TIMESTAMPTZ,
    last_predict_at TIMESTAMPTZ,
    last_eval_at TIMESTAMPTZ,
    status VARCHAR(20) DEFAULT 'idle',  -- idle, running, stale, error
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (symbol_id, interval_id)
);

CREATE TABLE sheets_sync_state (
    id SMALLINT PRIMARY KEY DEFAULT 1 CHECK (id = 1),
    last_synced_prediction_id BIGINT DEFAULT 0,
    last_synced_at TIMESTAMPTZ,
    last_stats_sync_at TIMESTAMPTZ,
    sync_status VARCHAR(20) DEFAULT 'idle',
    error_message TEXT
);
```

## DDL — Core (giữ nguyên + bổ sung)

```sql
CREATE TYPE trend_label AS ENUM ('bullish', 'bearish', 'sideway');
CREATE TYPE risk_level AS ENUM ('low', 'medium', 'high');
CREATE TYPE vol_level AS ENUM ('normal', 'high', 'extreme');

CREATE TABLE ml_models (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    model_type VARCHAR(30) NOT NULL,
    version INT NOT NULL,
    interval VARCHAR(5) NOT NULL,
    artifact_path TEXT NOT NULL,
    metrics JSONB NOT NULL DEFAULT '{}',
    is_active BOOLEAN DEFAULT FALSE,
    dataset_end TIMESTAMPTZ NOT NULL,  -- last kline used in train
    trained_at TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (name, version, interval)
);

-- ml_training_runs, ml_benchmark_results: unchanged

CREATE TABLE predictions (
    id BIGSERIAL PRIMARY KEY,
    model_id INT REFERENCES ml_models(id),
    symbol_id INT NOT NULL REFERENCES symbols(id),
    interval_id INT NOT NULL REFERENCES intervals(id),
    predicted_at TIMESTAMPTZ NOT NULL,
    target_time TIMESTAMPTZ NOT NULL,
    current_price NUMERIC(20,8) NOT NULL,
    predicted_price NUMERIC(20,8) NOT NULL,
    trend trend_label NOT NULL,
    confidence NUMERIC(5,2) NOT NULL CHECK (confidence >= 0 AND confidence <= 100),
    expected_move_pct NUMERIC(10,4) NOT NULL,
    risk_score risk_level NOT NULL,
    volatility_score vol_level NOT NULL,
    sheets_synced_at TIMESTAMPTZ,  -- null = chưa sync Sheets
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (model_id, symbol_id, interval_id, target_time)
);

CREATE INDEX idx_predictions_unsynced ON predictions (id) WHERE sheets_synced_at IS NULL;
CREATE INDEX idx_predictions_lookup ON predictions (symbol_id, interval_id, target_time DESC);

CREATE TABLE prediction_evaluations (
    id BIGSERIAL PRIMARY KEY,
    prediction_id BIGINT NOT NULL REFERENCES predictions(id) UNIQUE,
    actual_price NUMERIC(20,8),
    trend_correct BOOLEAN,
    direction_correct BOOLEAN,
    error_pct NUMERIC(10,4),
    evaluated_at TIMESTAMPTZ DEFAULT NOW()
);
```

## Views

```sql
CREATE VIEW v_predictions_dashboard AS
SELECT s.symbol, i.name AS interval, p.predicted_at, p.target_time,
       p.current_price, p.predicted_price, p.trend, p.confidence,
       p.expected_move_pct, p.risk_score, p.volatility_score,
       e.actual_price,
       CASE WHEN e.direction_correct THEN 'WIN' WHEN e.direction_correct IS FALSE THEN 'LOSS' END AS result,
       e.trend_correct, e.direction_correct, e.error_pct
FROM predictions p
JOIN symbols s ON s.id = p.symbol_id
JOIN intervals i ON i.id = p.interval_id
LEFT JOIN prediction_evaluations e ON e.prediction_id = p.id;

CREATE VIEW v_rolling_stats AS
SELECT s.symbol, i.name AS interval,
       COUNT(*) FILTER (WHERE e.evaluated_at > NOW() - INTERVAL '7 days') AS n_7d,
       AVG(CASE WHEN e.direction_correct THEN 1.0 ELSE 0.0 END) FILTER (WHERE e.evaluated_at > NOW() - INTERVAL '7 days') AS win_rate_7d,
       AVG(CASE WHEN e.trend_correct THEN 1.0 ELSE 0.0 END) FILTER (WHERE e.evaluated_at > NOW() - INTERVAL '7 days') AS trend_acc_7d,
       AVG(e.error_pct) FILTER (WHERE e.evaluated_at > NOW() - INTERVAL '7 days') AS avg_error_7d,
       AVG(p.confidence) FILTER (WHERE p.predicted_at > NOW() - INTERVAL '7 days') AS avg_conf_7d
FROM predictions p
JOIN symbols s ON s.id = p.symbol_id
JOIN intervals i ON i.id = p.interval_id
LEFT JOIN prediction_evaluations e ON e.prediction_id = p.id
GROUP BY s.symbol, i.name;
```

## Queries (runtime)

**Incremental klines:**

```sql
SELECT * FROM klines k
JOIN symbols s ON s.id = k.symbol_id
JOIN intervals i ON i.id = k.interval_id
WHERE s.symbol = :symbol AND i.name = '1m'
  AND k.open_time > :watermark
ORDER BY k.open_time;
```

**Unsynced predictions (Sheets):**

```sql
SELECT * FROM v_predictions_dashboard p
WHERE p.id > :last_synced_id OR p.evaluated_at > :last_synced_at;
-- Hoặc: WHERE sheets_synced_at IS NULL
```

Schema change → migration + cập nhật file này.
