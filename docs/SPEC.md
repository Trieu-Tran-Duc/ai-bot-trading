# SPEC — Requirements

**WHAT** hệ thống phải làm. **HOW** → ARCHITECTURE.md.

## Goal

Thu thập dữ liệu Binance 24/7, validate + normalize, lưu PostgreSQL làm **Single Source of Truth** cho AI sau này.

## Phase 1 — In scope

| ID | Requirement |
|----|-------------|
| FR-01 | Historical sync (REST), incremental, no duplicates |
| FR-02 | Real-time WebSocket, multi-symbol, auto-reconnect |
| FR-03 | Validate mọi message (schema, types, symbol, timestamp) |
| FR-04 | Normalize (UTC, field names, Decimal precision) |
| FR-05 | Persist PostgreSQL (atomic batch, dedupe) |
| FR-06 | Telegram: start/stop/error/reconnect |
| FR-07 | REST API: status, health, collector control |
| FR-08 | Config 100% từ env (symbols, intervals, credentials) |

## Out of scope (Phase 1)

AI/ML · Prediction · TA indicators · Trading · Backtesting · Dashboard analytics

## Business rules

| Rule | Mô tả |
|------|-------|
| BR-01 | Validate trước khi lưu |
| BR-02 | Normalize trước khi lưu |
| BR-03 | DB = SSOT; downstream không gọi Binance |
| BR-04 | Raw data immutable (không UPDATE historical) |
| BR-05 | Dedupe via unique constraint + `ON CONFLICT DO NOTHING` |
| BR-06 | Collector fail không crash toàn platform |
| BR-07 | Gap detection → backfill, không fabricate data |

## NFR

| ID | Target |
|----|--------|
| NFR-01 | 24/7 uptime, auto-recover |
| NFR-02 | 0 intentional data loss |
| NFR-03 | Latency ghi DB < 500ms |
| NFR-04 | Secrets không trong code/logs |
| NFR-05 | Structured logs, traceable errors |

## Defaults v1

**Symbols:** BTCUSDT, ETHUSDT, SOLUSDT, BNBUSDT (config `SYMBOLS`)  
**Data:** Kline 1m + AggTrade (mở rộng qua config)  
**Exchange:** Binance Spot only

## Definition of Done — Phase 1

- [ ] Historical + realtime data cho 4 symbols
- [ ] Reconnect + gap backfill verified
- [ ] Telegram alerts hoạt động
- [ ] REST health/status OK
- [ ] 24h soak test stable
- [ ] Không code ML/trading trong repo

## Phase 2 (planned)

AI Research Platform — prediction, benchmark, confidence, continuous learning.  
→ [PHASE2-INDEX.md](PHASE2-INDEX.md) · [PHASE2-SPEC.md](PHASE2-SPEC.md)
