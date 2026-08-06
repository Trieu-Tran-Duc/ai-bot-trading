# AGENTS.md — AI Operating Rules

## Read order (mỗi task)

1. `docs/CONTEXT.md` → phase & priority hiện tại
2. **Phase 1:** SPEC / ARCHITECTURE / DATABASE / INTEGRATIONS (chọn file liên quan)
3. **Phase 2:** `docs/PHASE2-INDEX.md` → @ thêm 1–2 file theo attach matrix  
   - Jobs/train/predict/Sheets → **`PHASE2-RUNTIME.md`**
4. `docs/DECISIONS.md` nếu chạm stack hoặc kiến trúc
5. `.cursor/rules/*.mdc`

**Không** đọc lại toàn bộ docs. **Không** suy đoán ngoài tài liệu.

## Phase 1 scope

Thu thập Binance (REST + WebSocket) → validate → normalize → PostgreSQL → Telegram + REST API quản lý.

## Phase 2 scope

AI Research Platform: preparation → features → train → benchmark → predict → confidence → eval → retrain → Sheets.  
Chi tiết runtime: `docs/PHASE2-RUNTIME.md`. Chỉ đọc PostgreSQL (ADR-005).

## Non-goals (Phase 1)

Không implement: ML, prediction, TA indicators, trading, backtesting, dashboard AI.

## Non-goals (Phase 2)

Auto-trading · Binance access · Web dashboard UI

## Module boundaries

```
API → Service → Repository → DB
Collector → Validator → Normalizer → Writer → Repository
```

- Collector/Validator/Normalizer **không** gọi DB trực tiếp
- API **không** chứa business logic hay SQL
- AI modules tương lai **chỉ** đọc PostgreSQL

## Priority khi conflict

1. User instruction
2. `docs/DECISIONS.md`
3. `docs/SPEC.md`
4. `docs/ARCHITECTURE.md`
5. `docs/DATABASE.md`
6. `.cursor/rules/`

## Sau mỗi task

- Cập nhật `docs/CONTEXT.md` (phase, done, next)
- Cập nhật doc nếu đổi schema/architecture
- Minimal diff, không over-engineer

## Commit

Conventional: `feat:` `fix:` `docs:` `test:`
