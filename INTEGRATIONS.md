# INTEGRATIONS — Binance & Telegram

External API specs. Kiến trúc tổng → ARCHITECTURE.md.

---

## Binance Spot

### Endpoints

| Type | Base | Use |
|------|------|-----|
| WebSocket | `wss://stream.binance.com:9443` | Real-time |
| REST | `https://api.binance.com` | Backfill, validate |

**Public market data — không cần API key Phase 1.**

### WebSocket — combined stream

```
/stream?streams=btcusdt@kline_1m/btcusdt@aggTrade/ethusdt@kline_1m/...
```

Stream name **lowercase**. REST symbol **UPPERCASE**.

| Stream | Persist rule |
|--------|--------------|
| `@kline_{interval}` | Chỉ khi `k.x == true` |
| `@aggTrade` | Mọi event, dedupe by `agg_trade_id` |

### Kline fields → DB

`k.t`→open_time · `k.T`→close_time · `k.o/h/l/c/v/q/n` → OHLCV · ms → UTC

### REST backfill

```
GET /api/v3/klines?symbol=BTCUSDT&interval=1m&startTime={ms}&limit=1000
```

Chunk 1000, sleep 0.2s, max ~5 req/s. Weight: 2/request, budget 1200/min.

Also: `/api/v3/exchangeInfo` (validate symbols), `/api/v3/ping`

### Reconnect

Backoff 1s→60s (×2, jitter). Sau reconnect → backfill từ `MAX(open_time)`.

### Errors

| Code | Action |
|------|--------|
| Parse/validation fail | Log, skip message |
| WS closed | Reconnect + backfill |
| 429 | Backoff retry |
| 418 | CRITICAL, stop |

Testnet: `wss://testnet.binance.vision` (dev only)

---

## Telegram

### Setup

BotFather → `TELEGRAM_BOT_TOKEN` · getUpdates → `TELEGRAM_CHAT_ID`

### Events

| Event | Notify | Throttle |
|-------|--------|----------|
| COLLECTOR_START/STOP | ✅ | — |
| WS_DISCONNECTED | ✅ | 1/5min |
| BACKFILL_DONE | ✅ | — |
| HEALTH_CHECK_FAIL | ✅ | 1/10min |
| DB_ERROR | ✅ | — |
| Hourly digest | ✅ | config interval |

**Không notify:** mỗi kline/trade (trừ `TELEGRAM_NOTIFY_ON_KLINE=true`)

### Implementation

- `httpx.AsyncClient` → `api.telegram.org/bot{token}/sendMessage`
- Fire-and-forget: `asyncio.create_task(...)` — không block collector
- Failed send: retry 3× · log + ghi `audit_logs`

### Test

```bash
python scripts/test_telegram.py
```
