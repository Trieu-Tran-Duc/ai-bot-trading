# Cursor Rules

This file is a short operational guide for Cursor and AI coding agents.

## Read this first
- Read [docs/PROJECT_RULES.md](PROJECT_RULES.md) for the canonical project contract.
- Read [docs/BINANCE_DATA.md](BINANCE_DATA.md) for data source and schema expectations.
- Read [docs/REPO_CONVENTION_CHECKLIST.md](REPO_CONVENTION_CHECKLIST.md) before implementing or reviewing changes.

## Working rules
- Follow the existing architecture and layer boundaries.
- Prefer small, focused changes over large rewrites.
- Do not rename tables, schemas, or core components without explicit need.
- Do not add dependencies unless the project already uses them or they are clearly justified.
- Do not create placeholder code or TODOs.
- Keep code simple, readable, and production-ready.

## Implementation workflow
- Read the project rules and data docs before making changes.
- Keep the current phase scoped and avoid speculative features.
- Preserve existing patterns for collectors, services, repositories, and API layers.
- Verify that the change respects immutability, async design, and configuration rules.

## Implementation priorities
- Preserve immutability of raw market data.
- Prefer async, typed, and testable code.
- Use existing repository/service/collector patterns.
- Keep configuration in .env and avoid hardcoded values.

## Default implementation order
- Understand the requirement and affected layer.
- Follow the existing architecture boundaries.
- Implement the minimal change needed for the current phase.
- Add tests and update docs when behavior changes.
- Confirm the change still respects data and logging rules.
