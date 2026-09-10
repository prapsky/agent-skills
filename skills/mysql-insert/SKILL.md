---
name: mysql-insert
description: >-
  Generate or run safe local Docker MySQL seed SQL (DBeaver-ready), and ensure
  local Docker Redis is available for Playwright API tests. Use when seeding
  fixtures, mysql-insert, DBeaver SQL, or preparing data for Playwright local
  API tests. Prefer short verify JSON for downstream tools. Never use remote
  staging databases or staging Redis.
---

# MySQL insert

## Token rules (do these first)

1. **Reuse** prior seed IDs/paths under `/tmp/*-seed.json` when still valid — do not re-discover the whole schema.
2. **Do not** dump `.env` / passwords into chat or reports.
3. **One** discovery round → insert → verify; stop on success.
4. **Local Docker only** — MySQL (`app-mysql-local`) and Redis (`app-redis-local`). Never cloud DB proxies, remote staging DB/Redis, or ad-hoc tunnel ports used only for remote DBs.
5. Long SQL / Docker recipes → [reference.md](reference.md) only when needed.

## When to use / skip

| Use | Skip |
|-----|------|
| Seed / INSERT / DBeaver SQL against **local Docker MySQL** | Staging / production unless user explicitly asks |
| Prep data for Playwright local API (MySQL seed; Redis must be up) | PR create unless PR adds SQL seeds |
| Start / verify local Docker MySQL + Redis | Non-SQL stores (document DB, object storage, etc.) unless asked |

## Checklist

```
- [ ] 1. Docker daemon running (Docker Desktop or Colima)
- [ ] 2. Docker MySQL up on 127.0.0.1:3306 (app-mysql-local)
- [ ] 3. Docker Redis up on 127.0.0.1:6379 (app-redis-local) — required when pairing with Playwright
- [ ] 4. Target table + required columns (ask only if missing)
- [ ] 5. Discovery SELECT for FKs by name (no invented UUIDs)
- [ ] 6. INSERT (prefer INSERT…SELECT) + verify SELECT
- [ ] 7. Write seed JSON to /tmp/<case>-seed.json (no secrets)
- [ ] 8. Cleanup DELETE only if user asks
```

## Local Docker stack (required)

Before seeding or Playwright:

| Container | Port | Purpose |
|-----------|------|---------|
| `app-mysql-local` | `3306` | Seed + API MySQL |
| `app-redis-local` | `6379` | API cache (no password by default) |

Start commands and env alignment: [reference.md](reference.md).

Match the project’s local env file (e.g. `.env.testing` / `.env.local`) to this stack — do not point the API at remote staging for seed/test runs.

## Execute against local Docker MySQL

When the user asks to **run** seeds (not just print SQL):

| Step | Rule |
|------|------|
| DB | **Docker MySQL only** — `app-mysql-local` |
| Host/port | `127.0.0.1:3306` |
| Creds | Match container `MYSQL_*` and the project’s local env (defaults in [reference.md](reference.md)) |
| Env | Parse env file in Python — never `source` in zsh; do not use cloud instance connection names for local runs |
| Client | Homebrew `mysql` may fail auth plugin → `/tmp/.../venv` + **pymysql** |
| Schema | If DB is empty, apply schema + FK lookup rows before INSERT |
| Redis | Confirm `redis-cli ping` → `PONG` when work continues to Playwright |
| Output | Print verify row + path to seed JSON only |

### Forbidden (unless user explicitly overrides)

- Cloud SQL / managed-DB proxies aimed at remote staging
- Staging / production MySQL or Redis hosts
- Seeding against remote staging IPs

## Domain seed notes

Keep seed recipes **project-specific** in chat or a short case note — do not hard-code one company’s table names here.

General rules:

- Discover FK ids with `SELECT … WHERE name = …` (or the project’s real unique keys).
- Prefer `INSERT … SELECT` so FK lookups stay correct.
- Mark seed rows with a clear `remarks` / tag like `seed:<ticket-or-case>` when the schema has such a column.
- Only `DELETE` seed rows when the user asks.

## Output shape (chat)

1. One line: what was seeded  
2. Seed JSON path + key fields  
3. Cleanup SQL (only if useful)

Details: [reference.md](reference.md).
