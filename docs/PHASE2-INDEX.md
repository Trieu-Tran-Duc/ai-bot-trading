# Phase 2 — Doc Router

> Cursor: @ attach **2–3 file** theo bảng — không attach toàn bộ docs.

**Vision:** AI Research Platform — dự báo BTC, confidence, benchmark, continuous learning.  
**Input:** PostgreSQL realtime (Phase 1). **Không** gọi Binance (ADR-005).

## Attach matrix

| Task area | @ files |
|-----------|---------|
| Bất kỳ Phase 2 | `CONTEXT.md` + file này |
| Requirements / metrics / labels | `PHASE2-SPEC.md` |
| Pipeline / modules / layout | `PHASE2-ARCHITECTURE.md` |
| **Jobs / train / predict / data freshness** | **`PHASE2-RUNTIME.md`** |
| Schema / migrations | `PHASE2-DATABASE.md` |
| Google Sheets / Telegram | `PHASE2-INTEGRATIONS.md` + `PHASE2-RUNTIME.md` |
| Stack / ADR | `DECISIONS.md` |

## Runtime tóm tắt (1 dòng)

Candle close → incremental features → predict → evaluate → **Sheets sync ngay** → retrain theo lịch/perf.

→ Chi tiết: [PHASE2-RUNTIME.md](PHASE2-RUNTIME.md)

## Implementation order

→ [WORKFLOW.md](WORKFLOW.md#phase-2--implementation-order)

| # | Module | Doc chính |
|---|--------|-----------|
| 1 | Scaffold `app/ai/` | ARCHITECTURE |
| 2 | DB migration + pipeline state | DATABASE + RUNTIME |
| 3 | Data readiness + incremental load | **RUNTIME** |
| 4 | Preparation + resample multi-TF | SPEC + RUNTIME |
| 5 | Feature engineering | SPEC + ARCHITECTURE |
| 6 | Training + walk-forward benchmark | SPEC + RUNTIME |
| 7 | Predict + confidence | SPEC + RUNTIME |
| 8 | Evaluate + continuous learning | RUNTIME + DATABASE |
| 9 | Google Sheets event sync | INTEGRATIONS + RUNTIME |
| 10 | Telegram alerts | INTEGRATIONS |

## Non-goals

Auto-trading · Binance access · Web dashboard UI

## Success criteria

→ [PHASE2-SPEC.md](PHASE2-SPEC.md#definition-of-done)
