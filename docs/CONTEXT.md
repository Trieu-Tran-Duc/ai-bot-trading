# CONTEXT — Project Status

> Cập nhật mỗi session. Cursor đọc **đầu tiên**.

**Updated:** 2026-08-06 · **Phase:** 1 — AI Data Collector · **Status:** 🟡 Docs done, code chưa bắt đầu

## Done

- [x] Bộ docs Phase 1 (SPEC, ARCHITECTURE, DATABASE, INTEGRATIONS, WORKFLOW, DECISIONS)
- [x] Bộ docs Phase 2 (INDEX, SPEC, ARCHITECTURE, DATABASE, INTEGRATIONS, **RUNTIME**)
- [x] Cursor rules (tech-stack, coding-standard)

## Next (theo WORKFLOW.md)

1. [ ] Project scaffold (`app/`, pyproject, Docker)
2. [ ] Config + logging
3. [ ] DB models + Alembic migration
4. [ ] Repository layer
5. [ ] Binance REST collector + backfill

## Blockers

_None_

## Env

| Item | Status |
|------|--------|
| `.env` | Chưa tạo |
| PostgreSQL | Chưa chạy |
| Telegram bot | Chưa setup |

## Next Cursor prompt

```
Đọc docs/CONTEXT.md + WORKFLOW.md (Phase 1, bước 1–3).
Scaffold app/: pyproject.toml, config, logging, SQLAlchemy models, Alembic.
Theo ARCHITECTURE.md layout. Không implement Binance yet.
Cập nhật CONTEXT.md khi xong.
```

## Phase 2 (planned — chưa implement)

**Vision:** AI Research Platform — dự báo BTC, data luôn mới, Sheets sync realtime.  
**Doc router:** `docs/PHASE2-INDEX.md` · **Runtime:** `docs/PHASE2-RUNTIME.md`

| Status | Item |
|--------|------|
| 🟢 | Docs Phase 2 |
| ⬜ | Code `app/ai/` |

## Constraints

Binance only · PostgreSQL only · Python 3.13+ · Async I/O · Config từ `.env`
