# AI Crypto Data & Research Platform

Hệ thống thu thập dữ liệu Binance 24/7, lưu PostgreSQL (SSOT), và (Phase 2) AI Research Platform dự báo xu hướng Bitcoin với đánh giá mô hình chặt chẽ.

**Phase hiện tại:** 1 — AI Data Collector · Docs done, code chưa bắt đầu

## Vision

```text
Phase 1 (Collector)          Phase 2 (AI Research)
Binance → validate → DB  →   features → train → predict → evaluate → Sheets
        ↓ chạy song song ↓              ↑ đọc DB only, không gọi Binance
   PostgreSQL (SSOT)
```

| Phase | Mục tiêu | Status |
|-------|---------|--------|
| **1** | Thu thập realtime + historical → PostgreSQL | 🟡 Docs ✓ · Code ⬜ |
| **2** | AI train/predict/evaluate, confidence, Google Sheets | 🟢 Docs ✓ · Code ⬜ |

Phase 2 **không** auto-trading. AI chỉ đọc dữ liệu từ Database.

## Doc map

### Luôn đọc trước

| File | Nội dung |
|------|----------|
| [AGENTS.md](AGENTS.md) | Quy tắc Cursor, thứ tự đọc |
| [docs/CONTEXT.md](docs/CONTEXT.md) | **Trạng thái project** — cập nhật mỗi session |

### Phase 1 — Data Collector

| File | Nội dung |
|------|----------|
| [docs/SPEC.md](docs/SPEC.md) | Requirements, DoD |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Components, flow, layout |
| [docs/DATABASE.md](docs/DATABASE.md) | Schema klines, agg_trades |
| [docs/INTEGRATIONS.md](docs/INTEGRATIONS.md) | Binance + Telegram |
| [docs/WORKFLOW.md](docs/WORKFLOW.md) | Implementation order + Cursor prompts |
| [docs/DECISIONS.md](docs/DECISIONS.md) | ADR — không override |

### Phase 2 — AI Research Platform

| File | Nội dung |
|------|----------|
| [docs/PHASE2-INDEX.md](docs/PHASE2-INDEX.md) | **Router** — @ attach 2–3 files/task |
| [docs/PHASE2-RUNTIME.md](docs/PHASE2-RUNTIME.md) | Jobs, data freshness, train/predict cycle |
| [docs/PHASE2-SPEC.md](docs/PHASE2-SPEC.md) | Targets, metrics, confidence |
| [docs/PHASE2-ARCHITECTURE.md](docs/PHASE2-ARCHITECTURE.md) | AI pipeline, modules |
| [docs/PHASE2-DATABASE.md](docs/PHASE2-DATABASE.md) | predictions, ml_models, pipeline state |
| [docs/PHASE2-INTEGRATIONS.md](docs/PHASE2-INTEGRATIONS.md) | Google Sheets + Telegram AI |

## Stack

Python 3.13+ · FastAPI · SQLAlchemy 2 async · PostgreSQL · Alembic · httpx · websockets · APScheduler

Phase 2 thêm: XGBoost / LightGBM / PyTorch · Google Sheets API · scikit-learn

Chi tiết: [docs/DECISIONS.md](docs/DECISIONS.md)

## Quick start (sau khi implement Phase 1)

```bash
cp .env.example .env
docker compose up -d
alembic upgrade head
uvicorn app.main:app --reload
```

## Defaults

**Symbols:** BTCUSDT, ETHUSDT, SOLUSDT, BNBUSDT  
**Collector:** Kline 1m + AggTrade (Binance Spot)  
**Phase 2 predict TF:** 5m, 15m, 1h, 4h (resample từ 1m)

## License

See [LICENSE](LICENSE).
