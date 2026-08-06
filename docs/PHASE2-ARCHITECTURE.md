# Phase 2 — ARCHITECTURE

**HOW** AI pipeline. SPEC → PHASE2-SPEC.md · **Runtime/jobs** → PHASE2-RUNTIME.md.

## Pipeline

```text
PostgreSQL (Phase 1 — realtime ingest)
    │
    ▼ incremental read (watermark)
Data Preparation ──► resample 1m → TF ──► Feature Engineering
    │                                              │
    └──────────────────┬───────────────────────────┘
                       ▼
              Model Training ◄── retrain (latest data)
                       │
                       ▼
              Walk-forward Benchmark
                       │
                       ▼
              Prediction Engine (post candle close)
                       │
                       ▼
              Confidence Scoring
                       │
                       ▼
              Prediction DB ──► Evaluate ──► Sheets (event) / Telegram
```

## Module boundaries

| Module | Responsibility | Không được |
|--------|----------------|------------|
| `ai/data/` | Watermark, incremental kline load, stale check | Binance |
| `ai/preparation/` | Clean, resample, multi-TF align | train, predict |
| `ai/features/` | Features at t using data ≤ t only | persist klines |
| `ai/training/` | Fit, save artifacts, register | stale snapshot |
| `ai/evaluation/` | Metrics, benchmark, model rank | mutate predictions |
| `ai/prediction/` | Infer next-candle target | SQL outside repo |
| `ai/confidence/` | confidence/risk/vol scores | train |
| `ai/repository/` | AI DB CRUD | business logic in SQL |
| `ai/jobs/` | Orchestrator + scheduled jobs | block Phase 1 |

**Cấm:** ai/* → Binance · ai/* → UPDATE klines/agg_trades

## Folder layout

```text
app/ai/
├── data/                 # incremental loader, watermark
├── preparation/
├── features/
├── training/
│   ├── registry.py
│   └── trainers/
├── evaluation/
├── prediction/
├── confidence/
├── repository/
├── jobs/
│   ├── orchestrator.py
│   ├── data_ready.py
│   ├── feature_refresh.py
│   ├── predict_job.py
│   ├── evaluate_job.py
│   ├── retrain_job.py
│   └── sheets_sync_job.py
└── schemas/
models/                   # gitignored
```

## Model strategy

1. Train candidates on same walk-forward folds
2. Rank by composite score (SPEC)
3. Active: best single **or** weighted ensemble
4. Confidence uses ensemble agreement when `ENSEMBLE_ENABLED=true`

## Key flows

→ Chi tiết timing/trigger: [PHASE2-RUNTIME.md](PHASE2-RUNTIME.md)

**Incremental load:** watermark → fetch new klines → resample → update watermark

**Train:** full lookback → latest MAX(open_time) → walk-forward → register → atomic swap active

**Predict cycle:** data_ready → features → infer → confidence → insert → sheets sync

**Evaluate:** target_time passed → actual from klines → score → sheets update eval cols

## Error strategy

| Type | Action |
|------|--------|
| Stale/gap data | Skip predict, alert, no fake features |
| Train fail | Keep active model |
| Predict fail | Retry 1× next cycle, Telegram |
| Sheets fail | Retry 3×, cron backup, DB still SSOT |

Schema → [PHASE2-DATABASE.md](PHASE2-DATABASE.md)  
Sheets → [PHASE2-INTEGRATIONS.md](PHASE2-INTEGRATIONS.md)
