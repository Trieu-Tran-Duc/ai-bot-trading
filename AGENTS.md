# AGENTS.md — AI Operating Rules

## Read order (mỗi task)

1. `docs/CONTEXT.md` → phase & priority hiện tại
2. Doc liên quan task (SPEC / ARCHITECTURE / DATABASE / INTEGRATIONS)
3. `docs/DECISIONS.md` nếu chạm stack hoặc kiến trúc
4. `.cursor/rules/*.mdc`

**Không** đọc lại toàn bộ docs. **Không** suy đoán ngoài tài liệu.

## Phase 1 scope

Thu thập Binance (REST + WebSocket) → validate → normalize → PostgreSQL → Telegram + REST API quản lý.

## Non-goals (Phase 1)

Không implement: ML, prediction, TA indicators, trading, backtesting, dashboard AI.

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
