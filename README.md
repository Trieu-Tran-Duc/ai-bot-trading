# Binance AI Data Collector

Binance AI Data Collector is a Python-based data platform for collecting, validating, normalizing, and persisting Binance market data for future analysis and AI-oriented use cases.

## Overview

This project focuses on building a reliable, async-first pipeline for Binance market data with clear separation between collection, validation, persistence, and monitoring.

The system is designed around the following goals:
- Collect historical and real-time Binance market data
- Validate and normalize incoming payloads
- Store immutable raw market data
- Expose monitoring endpoints and notifications
- Keep the architecture modular and production-oriented

## Scope

### In scope
- Binance REST and WebSocket data collection
- Data validation and normalization
- Immutable raw-data persistence
- Monitoring APIs and Telegram notifications

### Out of scope
- Trading
- Prediction or ML models
- Technical indicators
- Strategy generation
- Feature engineering

## Architecture at a glance

The repository is structured around a simple layered design:
- Collector: acquires Binance data from REST and WebSocket sources
- Pipeline: validates and normalizes payloads
- Database writer / repository: persists data safely and efficiently
- API / Telegram: exposes monitoring and notifications

The design favors:
- Async I/O
- Type hints
- SQLAlchemy ORM
- Configuration through environment variables
- Immutable raw market data

## Repository structure

- docs/: project documentation, architecture, data rules, roadmap, and conventions
- AGENTS.md: instructions for agents and AI coding tools
- README.md: project overview and entry point

## Documentation

The project documentation is organized as follows:
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md): system design and component boundaries
- [docs/BINANCE_DATA.md](docs/BINANCE_DATA.md): Binance data sources and data rules
- [docs/CURSOR_RULES.md](docs/CURSOR_RULES.md): short guidance for Cursor and AI coding agents
- [docs/PROJECT_RULES.md](docs/PROJECT_RULES.md): canonical project rules
- [docs/REPO_CONVENTION_CHECKLIST.md](docs/REPO_CONVENTION_CHECKLIST.md): implementation checklist for contributors
- [docs/ROADMAP.md](docs/ROADMAP.md): phased implementation plan

## Development principles

The project follows these core rules:
- Keep modules small and focused on one responsibility
- Prefer editing existing modules over creating new ones
- Use configuration from .env rather than hardcoding values
- Use structured logging instead of print statements
- Preserve the immutability of raw market data
- Keep changes minimal, readable, and production-ready

## Getting started

The project is currently being organized around its documentation and architecture foundation. Implementation will follow the roadmap in [docs/ROADMAP.md](docs/ROADMAP.md).

To begin contributing:
1. Review the project rules in [docs/PROJECT_RULES.md](docs/PROJECT_RULES.md)
2. Read the architecture overview in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
3. Follow the repository checklist in [docs/REPO_CONVENTION_CHECKLIST.md](docs/REPO_CONVENTION_CHECKLIST.md)
4. Implement changes in small, focused steps

## Status

This repository currently provides the project foundation, documentation, and conventions needed for implementation. The next steps are to build the core collectors, pipeline, database layer, and monitoring components.
