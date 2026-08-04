# DECISIONS — ADR

Accepted decisions. **Không override** trừ khi user yêu cầu + thêm ADR mới.

| ID | Decision | Rejected |
|----|----------|----------|
| ADR-001 | **FastAPI** backend | Flask, Django |
| ADR-002 | **PostgreSQL** only | SQLite, MongoDB, MySQL |
| ADR-003 | **SQLAlchemy 2.x** async ORM | Peewee, Django ORM |
| ADR-004 | **Async-first** all external I/O | sync requests, threading |
| ADR-005 | **DB = SSOT** — AI không gọi Binance | direct exchange access |
| ADR-006 | **Immutable raw** market data | UPDATE historical rows |
| ADR-007 | **Layered**: API→Service→Repository→DB | skip layers |
| ADR-008 | **Repository pattern** — no direct SQL outside repo | raw SQL in API |
| ADR-009 | **Config from `.env`** only | hardcoded secrets |
| ADR-010 | **Docs authoritative** — update khi drift | silent divergence |

## ADR template (mới)

```markdown
## ADR-XXX: Title | Accepted | YYYY-MM-DD
Decision: ... | Context: ... | Rejected: ...
```

Deprecated → mark Superseded, không xóa ADR cũ.

## Priority

User > DECISIONS > SPEC > ARCHITECTURE > DATABASE > rules/
