# Repo Convention Checklist

Use this checklist before implementing or submitting changes.

## Scope and architecture
- [ ] Change stays within the project scope.
- [ ] Existing architecture and layer boundaries are preserved.
- [ ] No new module is introduced unless necessary.
- [ ] Existing components are preferred over creating new ones.

## Python and code quality
- [ ] Code uses Python 3.13+ conventions.
- [ ] Async patterns are used where appropriate.
- [ ] Type hints are present for public functions and methods.
- [ ] Logging is used instead of print().
- [ ] Code is small, focused, and readable.
- [ ] No placeholder code or TODOs unless explicitly requested.

## Data and persistence
- [ ] Raw Binance market data remains immutable.
- [ ] Raw data is stored before processing.
- [ ] Deduplication is handled before persistence.
- [ ] Timestamps are normalized to UTC and Binance event time is preserved.

## Configuration and dependencies
- [ ] Configuration is read from .env only.
- [ ] No hardcoded values that should be configured.
- [ ] No unnecessary dependencies are added.
- [ ] Existing dependency patterns are followed.

## Naming and structure
- [ ] Naming follows existing project conventions.
- [ ] Files and modules remain consistent with the current structure.
- [ ] Changes are minimal and production-ready.
