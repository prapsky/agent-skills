---
name: local-docker-mysql
description: >-
  Start, health-check, seed (INSERT…SELECT / DBeaver-ready), verify, and inspect
  local Docker MySQL for API and integration tests. Use when seeding fixtures,
  mysql-insert-style SQL, local MySQL, ticket-scoped data checks, or preflight
  before Playwright. Prefer short verify JSON for downstream tools. When local
  API testing follows, hand off to python-local-api-test for the mandatory
  result report. Never use remote staging or production MySQL unless the user
  explicitly overrides. (Replaces the former mysql-insert skill.)
---

# Local Docker MySQL

## Token rules (do these first)

1. **Reuse** prior seed IDs/paths under `/tmp/*-seed.json` when still valid — do not re-discover the whole schema.
2. **Do not** dump DB passwords into chat, HTML reports, or PR/issue comments.
3. **One** discovery round → insert → verify; stop on success.
4. **Local Docker only** — default MySQL `app-mysql-local` on `127.0.0.1:3306`. Pair with `local-docker-redis` (`app-redis-local`) when API needs cache. Never cloud DB proxies or remote staging/prod.
5. Parse local env files with **Python** — never `source` in zsh.
6. **New phones / string IDs** → follow [`test-data-conventions`](../test-data-conventions/SKILL.md) (`+6285YYMMDDxxx`, valid UUID PKs; discover FKs by name).
7. Long Docker / DBeaver / SQL recipes → [reference.md](reference.md).

## When to use / skip

| Use | Skip |
|-----|------|
| Start / ping / inspect local MySQL | Unit tests with mocks only |
| Seed / INSERT / DBeaver SQL against **local Docker MySQL** | Staging / production unless user explicitly asks |
| Prep data for Python local API | PR create unless PR adds SQL/Docker seed docs |
| Ticket-scoped `SELECT` across seed tags | Non-SQL stores → other `local-docker-*` skills |

## Defaults (rename per project — keep Docker and `.env*` in sync)

| Item | Default |
|------|---------|
| Container | `app-mysql-local` |
| Port | `3306` |
| Database | `app_local` |
| User / password | `app` / `app_local` |
| Root password | `app_root_local` |

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. Docker MySQL up on 127.0.0.1:3306 (app-mysql-local)
- [ ] 3. Docker Redis up when pairing with Playwright → follow local-docker-redis
- [ ] 4. Align service local env to 127.0.0.1:3306 (match MYSQL_*)
- [ ] 5. Target table + required columns (ask only if missing)
- [ ] 6. Discovery SELECT for FKs by real unique keys (do not invent FK UUIDs)
- [ ] 7. New string PKs = valid UUIDs; new phones = +6285YYMMDDxxx (or 6285… without + when the store requires it)
- [ ] 8. INSERT (prefer INSERT…SELECT) + verify SELECT
- [ ] 9. Write /tmp/<case>-seed.json (no secrets)
- [ ] 10. Cleanup DELETE only if user asks
```

## Start + health

```bash
docker start app-mysql-local 2>/dev/null || \
docker run -d --name app-mysql-local \
  -e MYSQL_DATABASE=app_local \
  -e MYSQL_USER=app \
  -e MYSQL_PASSWORD=app_local \
  -e MYSQL_ROOT_PASSWORD=app_root_local \
  -p 3306:3306 \
  mysql:8.0 \
  --default-authentication-plugin=mysql_native_password

docker exec app-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -papp_root_local --silent
```

Match the project’s local env (`.env.testing` / `.env.local`) to this container — do not point the API at remote staging for seed/test runs.

## Execute seeds (when user asks to run, not only print SQL)

| Step | Rule |
|------|------|
| DB | **Docker MySQL only** — `app-mysql-local` |
| Host/port | `127.0.0.1:3306` |
| Creds | Match container `MYSQL_*` and local env (defaults above) |
| Env | Parse env file in Python — never `source` in zsh; clear cloud instance connection names for local runs |
| Client | Homebrew `mysql` may fail auth plugin → `/tmp/.../venv` + **pymysql** |
| Schema | If DB is empty, apply schema + FK lookup rows before INSERT |
| Redis | Confirm `redis-cli ping` → `PONG` via `local-docker-redis` when continuing to Playwright |
| Output | Print verify row + path to seed JSON only |

### Seed rules

Keep table/column recipes **project-specific** in chat or a short case note — do not hard-code one company’s schema here.

- Discover FK ids with `SELECT … WHERE <unique_name_col> = …`.
- Prefer `INSERT … SELECT` so FK lookups stay correct.
- New primary-key string ids and phones → [`test-data-conventions`](../test-data-conventions/SKILL.md).
- Tag rows (`remarks` / equivalent) as `seed:<ticket-or-case>` when the schema allows.
- Only `DELETE` seed rows when the user asks.

### Ticket-scoped inspect

```bash
docker exec app-mysql-local mysql -uapp -papp_local app_local -e \
  "SELECT * FROM <table> WHERE remarks LIKE 'seed:<TICKET>%' OR id LIKE '<ticket>%' LIMIT 100;"
```

## Output shape (chat)

1. One line: what was seeded  
2. Seed JSON path + key fields  
3. Cleanup SQL (only if useful)

When work continues into a **local API** run, follow `python-local-api-test` and **always** deliver that skill’s result report format (chat + `result.html`; auto PR comment when a PR is in context).

## Related skills

| Skill | Role |
|-------|------|
| `test-data-conventions` | Phone `+6285YYMMDDxxx` + UUID string IDs |
| `local-docker-redis` | Cache companion for API tests |
| `local-docker-dynamodb` | Local Dynamo when the case needs it |
| `local-docker-firestore` | Firestore emulator when the case needs it |
| `python-local-api-test` | Local API probe + result report |

## Forbidden (unless user explicitly overrides)

- Cloud SQL / managed-DB proxies aimed at remote staging
- Staging / production MySQL hosts
- Seeding against remote staging IPs
- Committing real credentials into skill files or reports

Details: [reference.md](reference.md).
