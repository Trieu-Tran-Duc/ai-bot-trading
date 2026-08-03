# Project Rules

## Purpose
Binance AI Data Collector is a Python async platform for collecting, validating, normalizing, and persisting Binance market data for future analysis.

## In scope
- Historical and real-time Binance data collection
- Validation and normalization
- Immutable raw-data persistence
- Monitoring APIs and Telegram notifications

## Out of scope
- Trading
- Prediction or ML
- Technical indicators
- Strategy generation
- Feature engineering

## Hard rules
- Python 3.13+
- Prefer async I/O and type hints
- Use SQLAlchemy ORM
- Read configuration only from .env
- Use structured logging; never use print()
- Keep modules small and focused on one responsibility
- Prefer editing existing modules over creating new ones
- Do not change architecture unless absolutely necessary
- Do not hardcode values that belong in configuration
- Do not add unnecessary dependencies

## Data rules
- Market data is immutable
- Never update or delete raw market data
- Store raw data before processing
- Deduplicate before persistence
- Normalize timestamps to UTC and preserve the original Binance event time
- Keep raw payloads and normalized fields

## Architecture boundaries
- Collector: collects data only
- Pipeline/validator/normalizer: transforms data only
- Database writer/repository: persists data only
- API and Telegram: monitoring/notifications only; no direct Binance access
- AI modules: read from the database only

## Implementation expectations
- Follow existing naming conventions and layering
- Keep changes minimal, readable, and production-ready
- Avoid placeholders and TODOs unless explicitly requested
