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
| Schema | DATABASE.md |
| Architecture | ARCHITECTURE.md |
| API external | INTEGRATIONS.md |
| Status | CONTEXT.md |
| Stack/ADR | DECISIONS.md |

## Anti-patterns

| ❌ | ✅ |
|----|-----|
| "Build trading bot" | "Data collector only" |
| "Implement everything" | Một bước trong bảng trên |
| Không @ docs | @ 2–3 files liên quan |

## Future phases (reference)

2 Dataset · 3 Training · 4 Inference · 5 Signals — **ngoài scope**, chỉ ghi trong SPEC.
