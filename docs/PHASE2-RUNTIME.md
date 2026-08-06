# Phase 2 — Runtime & Data Freshness

> Đọc file này khi implement **jobs, train, predict, Sheets sync**.  
> Requirements → PHASE2-SPEC.md · Schema → PHASE2-DATABASE.md · Sheets API → PHASE2-INTEGRATIONS.md

## Nguyên tắc vàng

1. **Luôn đọc dữ liệu mới nhất** từ PostgreSQL (Phase 1) — incremental, không cache stale
2. **Chỉ dùng nến đã đóng** (`close_time <= now`) — không leak future data
3. **Predict ngay sau candle close** + buffer — không chờ cron dài
4. **Sheets sync ngay sau predict/eval** — cron chỉ là backup

## Orchestration (một chu kỳ)

```text
Phase 1 WS/REST → klines (closed) trong DB
        │
        ▼ (trigger: candle close + DATA_LAG_BUFFER_SEC)
┌───────────────────────────────────────────────────┐
│ 1. data_ready_check    — kline mới? gap?          │
│ 2. feature_refresh     — incremental từ watermark │
│ 3. predict_job         — infer + confidence       │
│ 4. evaluate_job        — score predictions đến hạn │
│ 5. sheets_sync_job     — incremental upsert       │
│ 6. retrain_job         — chỉ khi schedule/trigger │
└───────────────────────────────────────────────────┘
        │
        ▼
Google Sheets (Predictions + Stats) · Telegram summary
```

**Thứ tự bắt buộc:** data_ready → features → predict → evaluate → sheets. Retrain **không** chặn predict.

## Data freshness contract

| Khái niệm | Rule |
|-----------|------|
| Source of truth | `klines` + `agg_trades` (Phase 1) |
| Read mode | Incremental: `open_time > watermark` |
| Watermark | `ai_pipeline_state.last_processed_open_time` per (symbol, interval) |
| Resample | 1m → 5m/15m/1h/4h trong `preparation/` — luôn từ DB mới nhất |
| Gap | Gap > `MAX_GAP_INTERVALS` → skip predict, alert, **không** fabricate |
| Train dataset | `dataset_end = MAX(open_time)` lúc train start — luôn full history → latest |

### data_ready_check

Pass khi **tất cả** đúng:

- Có kline closed mới cho mỗi `(symbol, interval)` since last watermark
- `now - last_kline.close_time < STALE_DATA_THRESHOLD` (default 2× interval)
- Phase 1 collector_status = healthy (optional gate)

Fail → log + Telegram `AI_DATA_STALE`, skip predict, **không** train.

## Job schedule

| Job | Trigger | Default |
|-----|---------|---------|
| `feature_refresh` | Sau candle close | +30s buffer (`DATA_LAG_BUFFER_SEC`) |
| `predict_job` | Sau feature_refresh OK | cùng chu kỳ |
| `evaluate_job` | Sau predict + mỗi interval close | score rows có `target_time <= last_close` |
| `sheets_sync_job` | **Sau** predict/eval success | event-driven |
| `sheets_sync_job` | Cron backup | mỗi 5 phút (`SHEETS_SYNC_BACKUP_MIN`) |
| `retrain_job` | Cron + perf trigger | weekly + win rate drop |

Config intervals: `PREDICTION_INTERVALS=5m,15m,1h,4h`

## Training — dữ liệu luôn mới

```text
Load klines [now - TRAIN_LOOKBACK_DAYS .. MAX(open_time)]
  → resample multi-TF → features (no future leak)
  → walk-forward split → train candidates → benchmark
  → register best/ensemble → set is_active
```

| Param | Default | Mô tả |
|-------|---------|-------|
| `TRAIN_LOOKBACK_DAYS` | 180 | Min history |
| `WALK_FORWARD_TRAIN_DAYS` | 120 | Fold train window |
| `WALK_FORWARD_TEST_DAYS` | 14 | Fold test window |
| `WALK_FORWARD_EMBARGO` | 1 interval | Gap train/test chống leak |
| `MIN_TRAIN_SAMPLES` | 10000 | Dưới ngưỡng → skip train |

**Retrain trigger** (bất kỳ):

- Cron `RETRAIN_CRON` (default Sun 02:00 UTC)
- Rolling win rate < `RETRAIN_WIN_RATE_THRESHOLD` (default 52%)
- Model age > `RETRAIN_MAX_AGE_DAYS` (default 7)

Retrain luôn dùng data **đến kline mới nhất** — không snapshot cũ.

## Predict — timing

| Bước | Rule |
|------|------|
| Feature cutoff | Chỉ data với `open_time <= last_closed_candle` |
| Target | `target_time = last_closed_candle + 1 interval` |
| Current price | `close` của nến vừa đóng |
| Predicted price | Model output cho `target_time` |

## Env (runtime)

| Var | Default |
|-----|---------|
| `DATA_LAG_BUFFER_SEC` | 30 |
| `STALE_DATA_THRESHOLD` | auto (2× interval) |
| `MAX_GAP_INTERVALS` | 3 |
| `TRAIN_LOOKBACK_DAYS` | 180 |
| `RETRAIN_CRON` | `0 2 * * 0` |
| `RETRAIN_WIN_RATE_THRESHOLD` | 0.52 |
| `SHEETS_SYNC_BACKUP_MIN` | 5 |
| `SHEETS_SYNC_ON_EVENT` | true |

## Module map

| Job | Module |
|-----|--------|
| data_ready_check | `ai/jobs/data_ready.py` |
| feature_refresh | `ai/jobs/feature_refresh.py` |
| predict | `ai/jobs/predict_job.py` |
| evaluate | `ai/jobs/evaluate_job.py` |
| retrain | `ai/jobs/retrain_job.py` |
| sheets | `ai/jobs/sheets_sync_job.py` |
| orchestrator | `ai/jobs/orchestrator.py` |

Orchestrator: một APScheduler / asyncio loop — **không** Kafka v1.
