# AGENTS.md

# AGENTS.md

## Project context
This repository is a Binance AI Data Collector project for collecting, validating, normalizing, and persisting immutable Binance market data.

## Scope
In scope:
- Historical and real-time Binance data collection
- Validation and normalization
- Immutable raw-data persistence
- Monitoring APIs and Telegram notifications

Out of scope:
- Trading
- Prediction or ML
- Technical indicators
- Strategy generation
- Feature engineering

## Required reading
Before implementing anything, read these files first:
- docs/PROJECT_RULES.md
- docs/BINANCE_DATA.md
- docs/CURSOR_RULES.md
- docs/REPO_CONVENTION_CHECKLIST.md

## Core rules
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
- Normalize timestamps to UTC and preserve Binance event time

## Cursor guidance
- Follow the existing architecture and layering
- Keep changes minimal, readable, and production-ready
- Avoid placeholders and TODOs unless explicitly requested
