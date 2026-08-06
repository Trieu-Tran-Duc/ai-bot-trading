# Phase 2 — SPEC (Requirements)

**WHAT** AI Research Platform phải làm. **HOW** → PHASE2-ARCHITECTURE.md · **Runtime** → PHASE2-RUNTIME.md.

## Objective

Dự báo xu hướng BTC đa timeframe · confidence · tự đánh giá · continuous learning · Sheets dashboard realtime.

## In scope

| ID | Requirement |
|----|-------------|
| FR-P2-01 | Incremental data load từ DB — luôn đến kline mới nhất (watermark) |
| FR-P2-02 | Resample 1m → 5m/15m/1h/4h; align multi-TF; no future leak |
| FR-P2-03 | Feature engineering — không train raw OHLCV |
| FR-P2-04 | Walk-forward train + benchmark nhiều model; best hoặc ensemble |
| FR-P2-05 | Predict ngay sau candle close (+ buffer) |
| FR-P2-06 | Confidence = model prob + ensemble agreement + rolling perf + regime |
| FR-P2-07 | Risk + volatility score mỗi prediction |
| FR-P2-08 | Evaluate khi actual có (nến target đóng) |
| FR-P2-09 | **Sheets sync event-driven** sau predict/eval + cron backup |
| FR-P2-10 | Retrain scheduled + perf trigger — dataset luôn latest |
| FR-P2-11 | Telegram batch summary / stale data / retrain alerts |

## Out of scope

Auto-trading · Binance API · Web dashboard · Sub-ms inference

## Prediction targets (bắt buộc rõ)

Mỗi `(symbol, interval)` predict **một horizon** = 1 candle tiếp theo.

| Output | Label / cách tính |
|--------|-------------------|
| `predicted_price` | Regression: next candle **close** |
| `expected_move_pct` | `(predicted_price - current_price) / current_price × 100` |
| `trend` | `sideway` nếu \|move\| < `TREND_THRESHOLD_PCT` (default 0.15%) · else bullish/bearish |
| `current_price` | Close nến vừa đóng (feature cutoff) |

**Actual (eval):** close tại `target_time` từ `klines` — không nguồn khác.

## Timeframes

Default `5m,15m,1h,4h` — resample từ 1m Phase 1. Config `PREDICTION_INTERVALS`.

## Feature groups

| Group | Examples | Lookback gợi ý |
|-------|----------|----------------|
| Price | OHLCV, VWAP, returns | 1–60 bars |
| Trend | EMA, SMA, MACD, RSI, BB, ATR | 7–200 bars |
| Structure | HH, HL, LH, LL | swing window 5–20 |
| Order flow | buy/sell vol, delta *(agg_trades)* | rolling 15m |
| Volatility | rolling vol, realized vol | 14–30 bars |
| Momentum | ROC, Stoch RSI | 14 bars |
| Time | hour, DOW, session | — |
| Multi-TF | 1h trend khi predict 5m | higher TF ffilled |

Rule: mọi feature tại `t` chỉ dùng data `<= t` (ADR-012, ADR-016).

## Models (benchmark pool)

Tree: XGBoost, LightGBM, CatBoost · Sequence: LSTM, GRU · Attention: Transformer, TFT

Rank **primary:** direction accuracy + trend accuracy · **secondary:** MAE, F1.

Deploy: single best hoặc weighted ensemble (ADR-011). Weights từ inverse validation error.

## Evaluation metrics

**Trading (primary):** trend accuracy · direction accuracy · expected move error · rolling win rate (7d/30d)

**Secondary:** MAE, RMSE, MAPE · Accuracy, F1, ROC-AUC

**Model selection score:**

```text
score = 0.4×direction_acc + 0.35×trend_acc + 0.15×(1-normalized_MAE) + 0.10×F1
```

## Confidence composition

```text
confidence = 0.35×model_prob + 0.25×ensemble_agreement + 0.25×rolling_accuracy + 0.15×regime_stability
```

Clamp 0–100. `risk_score` ↑ khi volatility cao hoặc confidence thấp. `volatility_score` từ ATR percentile 30d.

## Prediction output

→ Schema: [PHASE2-DATABASE.md](PHASE2-DATABASE.md)

## Continuous learning

```text
Predict → candle close → actual from klines → score → DB → Sheets
  → rolling metrics → retrain if trigger (RUNTIME)
```

Track perf theo: interval · model · session · volatility regime.

## Google Sheets

Event sync sau predict/eval. Tabs: Predictions · Stats · Models.

→ [PHASE2-INTEGRATIONS.md](PHASE2-INTEGRATIONS.md)

## Business rules

| Rule | Mô tả |
|------|-------|
| BR-P2-01 | AI chỉ đọc PostgreSQL |
| BR-P2-02 | Không mutate Phase 1 tables |
| BR-P2-03 | Model version tracked; swap `is_active` atomically |
| BR-P2-04 | Walk-forward only — no random shuffle TS |
| BR-P2-05 | Prediction immutable; eval append-only |
| BR-P2-06 | Skip predict nếu data stale/gap — không dự đoán trên data cũ |
| BR-P2-07 | Sheets sync không được lag > `SHEETS_MAX_LAG_MIN` (default 10) sau predict |

## Definition of Done

- [ ] Incremental load + watermark; predict sau candle close verified
- [ ] Walk-forward train ≥2 model types; benchmark in DB
- [ ] Predictions + confidence + risk + vol multi-TF
- [ ] Eval loop: actual từ klines; rolling metrics
- [ ] Sheets event sync + Stats tab; lag < 10 min
- [ ] Retrain on schedule + perf trigger với latest data
- [ ] Không trading execution
