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

## Implementation workflow
- Read the relevant project docs before implementing or reviewing changes.
- Start from existing modules and keep changes scoped to the current task.
- Do not introduce new architectural patterns unless the current design requires it.
- Verify that the implementation preserves data immutability, boundary rules, and configuration conventions.

## Implementation expectations
- Follow existing naming conventions and layering
- Keep changes minimal, readable, and production-ready
- Avoid placeholders and TODOs unless explicitly requested

## Coding standards
- Follow PEP 8 style and keep code readable.
- Use type hints for public functions and methods.
- Prefer small, focused functions and classes.
- Avoid magic values; move constants to configuration.
- Keep imports explicit and avoid wildcard imports.

## Error handling strategy
- Catch external exceptions at integration boundaries.
- Retry transient REST failures with backoff.
- Reconnect WebSocket clients automatically and resubscribe safely.
- Log all failures with enough context for diagnosis and recovery.
- Do not let one failed symbol or stream stop unrelated processing.

## Logging specification
- Use structured logging with consistent fields such as event, module, symbol, and status.
- Never use print() for application logging.
- Log retries, validation failures, persistence events, and notification delivery.
- Avoid logging sensitive secrets or full credentials.

## Delivery checklist
- Requirements are covered by the current docs.
- The change does not break the existing architecture.
- The change remains consistent with data and persistence rules.
- The change is documented where necessary for future implementation.

## Implementation contract for contributors
- Implement only the current phase requirements.
- Keep changes small and targeted.
- Prefer extending existing modules rather than introducing new abstractions.
- Preserve existing interfaces unless the architecture explicitly requires a change.
- If a requirement is ambiguous, follow the existing architecture and document the assumption.
