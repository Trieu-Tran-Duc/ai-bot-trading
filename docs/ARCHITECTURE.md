# ARCHITECTURE

**HOW** hệ thống được xây. Business → SPEC.md.

## Flow

```text
Binance (REST/WS) → Collector → Validator → Normalizer → Writer → Repository → PostgreSQL
                                                                              ↓
                                                                    Telegram / REST API
Future AI ← chỉ đọc PostgreSQL
```

## Components

| Module | Responsibility | Không được |
|--------|----------------|------------|
| `collector/` | Lấy data Binance | validate, persist |
| `validator/` | Schema + business validation | DB, API |
| `normalizer/` | Binance → internal format | DB |
| `writer/` | Batch insert, retry | Gọi Binance |
| `repository/` | CRUD, queries | Business logic |
| `api/` | HTTP endpoints (thin) | SQL, business rules |
| `notification/` | Telegram alerts | Block collector |
| `config/` | Pydantic Settings | Hardcode |

Phase 1 queue: in-memory async (không Kafka/Celery).

## Dependencies (chỉ flow xuống)

```text
api → service → repository → database
collector → validator → normalizer → writer → repository
```

**Cấm:** collector→DB · api→DB · validator→repository

## Folder layout

```text
app/
├── api/              # FastAPI routers
├── collector/
│   ├── historical/   # REST backfill
│   └── realtime/     # WebSocket
├── validator/
├── normalizer/
├── writer/
├── repository/
├── database/         # models, session
├── schemas/          # Pydantic
├── notification/
├── config/
├── logging/
└── main.py
tests/unit/ · tests/integration/
```

## Key flows

**Realtime kline:** WS message → validate → normalize → persist **chỉ khi candle closed** (`k.x == true`)

**Reconnect:** disconnect → backoff reconnect → REST backfill gap từ `MAX(open_time)`

**Health (60s):** WS ok? · last kline fresh? · DB ping? → Telegram nếu fail

## Error strategy

| Type | Action |
|------|--------|
| Recoverable (timeout, WS drop, 429) | Retry/reconnect + log |
| Non-recoverable (bad config, schema) | Fail fast + Telegram |

## Deploy (Phase 1)

Docker Compose: FastAPI + PostgreSQL. Single host.

## AI contract (Phase 1)

AI đọc qua: SQL query · export script (Parquet). Không endpoint data API bắt buộc v1.

## Phase 2 — AI Research Platform

Pipeline + modules → [PHASE2-ARCHITECTURE.md](PHASE2-ARCHITECTURE.md)  
Runtime (jobs, freshness, Sheets trigger) → [PHASE2-RUNTIME.md](PHASE2-RUNTIME.md)  
Router → [PHASE2-INDEX.md](PHASE2-INDEX.md)

Chi tiết Binance/Telegram → [INTEGRATIONS.md](INTEGRATIONS.md)  
Schema → [DATABASE.md](DATABASE.md) · Phase 2 tables → [PHASE2-DATABASE.md](PHASE2-DATABASE.md)
