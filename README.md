# AI Crypto Data Platform

AI-First platform thu thập dữ liệu Binance → PostgreSQL (SSOT) → phục vụ AI Training/Inference sau này.

**Phase hiện tại:** AI Data Collector — chỉ Data Layer, không ML/trading.

## Doc map (đọc theo thứ tự)

| File | Nội dung |
|------|----------|
| [AGENTS.md](AGENTS.md) | Quy tắc AI, thứ tự đọc |
| [docs/CONTEXT.md](docs/CONTEXT.md) | **Trạng thái hiện tại** — đọc trước mỗi session |
| [docs/SPEC.md](docs/SPEC.md) | WHAT: scope, FR, NFR, DoD |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | HOW: components, flow, layout |
| [docs/DATABASE.md](docs/DATABASE.md) | Schema, insert rules |
| [docs/INTEGRATIONS.md](docs/INTEGRATIONS.md) | Binance + Telegram specs |
| [docs/WORKFLOW.md](docs/WORKFLOW.md) | Phases + Cursor prompts |
| [docs/DECISIONS.md](docs/DECISIONS.md) | ADR (không override) |

## Quick start (sau khi implement)

```bash
cp .env.example .env
docker compose up -d
alembic upgrade head
uvicorn app.main:app --reload
```

## Stack (tóm tắt)

Python 3.13 · FastAPI · SQLAlchemy 2 async · PostgreSQL · Alembic · httpx · websockets · APScheduler · Telegram

Chi tiết: `.cursor/rules/tech-stack.mdc`

## Roadmap (tham khảo)

| Phase | Nội dung |
|-------|----------|
| **1** | Data Collector ← **hiện tại** |
| 2 | Feature Engineering, Dataset |
| 3 | AI Training |
| 4 | Inference, Signals |
