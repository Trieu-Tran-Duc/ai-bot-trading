# WORKFLOW — Development & Cursor

## Trước mỗi task

1. Đọc `CONTEXT.md` (phase, priority)
2. @ attach doc liên quan — **không** attach toàn bộ docs
3. Nêu phase + scope cụ thể + non-goals

## Phase 1 — Implementation order

| # | Task | Acceptance |
|---|------|------------|
| 1 | Scaffold: `app/`, pyproject, Docker, `.env` | App starts |
| 2 | Config (Pydantic) + structured logging | Settings from env |
| 3 | DB models + Alembic migration | `alembic upgrade head` |
| 4 | Repository layer | CRUD klines tested |
| 5 | Binance REST collector + backfill | Historical data in DB |
| 6 | Validator + Normalizer (Pydantic) | Invalid data rejected |
| 7 | WebSocket collector + reconnect | Realtime klines 1m |
| 8 | Writer (batch insert) | Dedupe works |
| 9 | Telegram notification | test_telegram OK |
| 10 | REST API (health, status) | `/health` 200 |
| 11 | Integration test + 24h soak | Stable run |

**Không nhảy thứ tự.** Không implement Phase 2+ trừ khi được yêu cầu.

## Cursor prompt templates

### Scaffold (bước 1–3)

```
@docs/CONTEXT.md @docs/ARCHITECTURE.md @docs/DATABASE.md

Phase 1 bước 1–3: scaffold app/, config, logging, SQLAlchemy models, Alembic.
Không Binance yet. Cập nhật CONTEXT.md.
```

### Binance REST (bước 5)

```
@docs/INTEGRATIONS.md @docs/DATABASE.md

Implement historical collector + backfill. ON CONFLICT DO NOTHING.
Test BTCUSDT 1m, 1 ngày.
```

### WebSocket (bước 7)

```
@docs/INTEGRATIONS.md @docs/ARCHITECTURE.md

WS collector: combined stream, k.x filter, reconnect + gap backfill.
```

### Telegram (bước 9)

```
@docs/INTEGRATIONS.md

notification service: httpx, throttle, start/stop/error templates.
```

## Review checklist (cuối mỗi bước)

- [ ] SPEC non-goals không vi phạm?
- [ ] Module boundaries (ARCHITECTURE)?
- [ ] Dedupe + UTC + async I/O?
- [ ] CONTEXT.md updated?

## Doc update matrix

| Thay đổi | File |
|----------|------|
| Schema Phase 1 | DATABASE.md |
| Schema Phase 2 | PHASE2-DATABASE.md |
| Architecture Phase 2 | PHASE2-ARCHITECTURE.md |
| Runtime / jobs / freshness | PHASE2-RUNTIME.md |
| Architecture Phase 1 | ARCHITECTURE.md |
| API external | INTEGRATIONS.md · PHASE2-INTEGRATIONS.md |
| Phase 2 requirements | PHASE2-SPEC.md |
| Status | CONTEXT.md |
| Stack/ADR | DECISIONS.md |

## Phase 2 — Implementation order

**Prerequisite:** Phase 1 Done (1m klines realtime stable).

| # | Task | Acceptance | @ docs |
|---|------|------------|--------|
| 1 | Scaffold `app/ai/` + config | Import OK | INDEX + ARCHITECTURE |
| 2 | DB migration (incl. pipeline state) | `alembic upgrade head` | DATABASE + RUNTIME |
| 3 | Incremental loader + watermark | New klines since watermark | **RUNTIME** |
| 4 | data_ready + orchestrator | Cycle runs post candle close | **RUNTIME** |
| 5 | Preparation + resample multi-TF | 5m/15m/1h/4h from 1m | SPEC + RUNTIME |
| 6 | Feature engineering | No future leak tested | SPEC + ARCHITECTURE |
| 7 | Training + walk-forward benchmark | ≥2 models, composite score | SPEC + RUNTIME |
| 8 | Predict + confidence | Insert after candle close | SPEC + RUNTIME |
| 9 | Evaluate job | Actual from klines | RUNTIME + DATABASE |
| 10 | Sheets event sync | Lag < 10 min, incremental | INTEGRATIONS + RUNTIME |
| 11 | Retrain job | Latest data + perf trigger | RUNTIME |
| 12 | Telegram AI alerts | stale/sheets/retrain OK | INTEGRATIONS |

## Phase 2 — Cursor prompt templates

### Scaffold + pipeline state (bước 1–4)

```
@docs/PHASE2-RUNTIME.md @docs/PHASE2-DATABASE.md @docs/PHASE2-ARCHITECTURE.md

Phase 2 bước 1–4: app/ai/, migration ai_pipeline_state, incremental loader, orchestrator.
Post candle close cycle. Cập nhật CONTEXT.md.
```

### Feature + train (bước 5–7)

```
@docs/PHASE2-SPEC.md @docs/PHASE2-RUNTIME.md

Resample 1m→multi-TF, features, walk-forward train XGBoost+LSTM.
Composite score benchmark. dataset_end = latest kline.
```

### Predict + evaluate + Sheets (bước 8–10)

```
@docs/PHASE2-RUNTIME.md @docs/PHASE2-INTEGRATIONS.md @docs/PHASE2-DATABASE.md

Predict + confidence + evaluate. Sheets event sync incremental.
sheets_synced_at watermark. Không Binance.
```

### Retrain (bước 11)

```
@docs/PHASE2-RUNTIME.md @docs/PHASE2-SPEC.md

Retrain job: cron + win rate trigger. Atomic is_active swap.
```

## Anti-patterns

| ❌ | ✅ |
|----|-----|
| "Build trading bot" | "Data collector only" |
| "Implement everything" | Một bước trong bảng trên |
| Không @ docs | @ 2–3 files liên quan |
| @ toàn bộ docs/ | @ PHASE2-INDEX + 1–2 file theo task |
| Train trên Binance live | Đọc PostgreSQL only |

## Future phases (reference)

Phase 2 = AI Research Platform (unified). Xem PHASE2-INDEX.md — không còn tách Phase 3/4/5 riêng.
