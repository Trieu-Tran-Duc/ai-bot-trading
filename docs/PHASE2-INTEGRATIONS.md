# Phase 2 — INTEGRATIONS

Google Sheets + Telegram. Binance → Phase 1 only. **Runtime trigger** → [PHASE2-RUNTIME.md](PHASE2-RUNTIME.md).

## Google Sheets

### Role

Dashboard nghiên cứu realtime: predictions · actual · win rate · model rank.  
PostgreSQL = SSOT (ADR-014). Sheets = mirror cho phân tích.

### Setup (env)

| Var | Default | Mô tả |
|-----|---------|-------|
| `GOOGLE_SHEETS_CREDENTIALS` | — | Service account JSON path |
| `GOOGLE_SHEETS_SPREADSHEET_ID` | — | Spreadsheet ID |
| `SHEETS_SYNC_ON_EVENT` | true | Sync ngay sau predict/eval |
| `SHEETS_SYNC_BACKUP_MIN` | 5 | Cron backup nếu event miss |
| `SHEETS_MAX_LAG_MIN` | 10 | Alert nếu chậm hơn (BR-P2-07) |
| `SHEETS_STATS_WINDOW` | 500 | Rows cho rolling stats |
| `SHEETS_BATCH_SIZE` | 100 | Rows per API batch |

### Sheet tabs

| Tab | Nguồn | Update |
|-----|-------|--------|
| `Predictions` | `v_predictions_dashboard` | Incremental upsert mỗi event |
| `Stats` | `v_rolling_stats` | Full rewrite mỗi sync (nhỏ) |
| `Models` | `ml_models` + benchmark | On retrain / daily |

### Columns — Predictions

| Col | Field |
|-----|-------|
| A | predicted_at (UTC) |
| B | symbol |
| C | interval |
| D | current_price |
| E | predicted_price |
| F | trend |
| G | confidence % |
| H | expected_move_pct |
| I | risk_score |
| J | volatility_score |
| K | target_time |
| L | actual_price *(eval sau)* |
| M | result WIN/LOSS/PENDING |
| N | error_pct |

**Key upsert:** `(symbol, interval, target_time)` — Google Sheets `batchUpdate` match row hoặc append.

### Sync flow (event-driven)

```text
predict_job OK ──┐
evaluate_job OK ─┼──► sheets_sync_job
cron backup ─────┘
        │
        ▼
1. Load unsynced: sheets_synced_at IS NULL OR eval updated
2. Upsert Predictions tab (batch)
3. Rewrite Stats tab từ v_rolling_stats
4. Update Models tab if retrain since last sync
5. SET sheets_synced_at = NOW(), update sheets_sync_state
6. Alert if lag > SHEETS_MAX_LAG_MIN
```

### Sync rules

- **Incremental only** — không full dump mỗi lần
- Eval columns (L,M,N) update **in-place** cùng row prediction
- Fail: retry 3× exponential · predictions vẫn an toàn trong DB
- Fail persistent: Telegram `SHEETS_SYNC_FAIL`, cron retry

### Stats tab (computed)

| Metric | Nguồn |
|--------|-------|
| Win Rate 7d/30d | `v_rolling_stats` |
| Trend Accuracy | `v_rolling_stats` |
| Avg Error | `v_rolling_stats` |
| Avg Confidence | `v_rolling_stats` |
| Model Ranking | benchmark composite score |
| Profit Simulation | optional: sum signed expected_move when WIN |

### Implementation

- Google Sheets API v4 `spreadsheets.values.batchUpdate` via `httpx` (async)
- Service account share spreadsheet Editor
- Job: `ai/jobs/sheets_sync_job.py` — gọi từ orchestrator, không standalone cron-only

Test: `python scripts/test_sheets_sync.py`

---

## Telegram (Phase 2)

| Event | Khi nào | Throttle |
|-------|---------|----------|
| PREDICTION_BATCH_DONE | Sau predict + sheets OK | 1/interval |
| EVAL_BATCH_DONE | Sau evaluate + sheets OK | 1/interval |
| AI_DATA_STALE | data_ready fail | 1/hour |
| MODEL_RETRAIN_DONE/FAIL | Retrain xong | — |
| SHEETS_SYNC_FAIL | 3 retry fail | 1/hour |
| BENCHMARK_REGRESSION | Active model worse | 1/day |

Config: `TELEGRAM_AI_NOTIFY=true`

### Template

```
📊 BTCUSDT 1h close
Trend: Bullish 91% | Move: +1.4% | Risk: Low
Win 7d: 58% | Sheets: synced ✓
```
