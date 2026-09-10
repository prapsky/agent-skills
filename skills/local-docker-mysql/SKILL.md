---
name: local-docker-mysql
description: >-
  Start, health-check, seed, verify, and inspect local Docker MySQL for API and
  integration tests. Use when the user wants local MySQL, Docker MySQL seeding,
  ticket-scoped data checks, or preflight before Playwright. Prefer
  mysql-insert for INSERT…SELECT recipes. Never use remote staging or production
  MySQL unless the user explicitly overrides.
---

# Local Docker MySQL

## Token rules (do these first)

1. **Local Docker only** — default container `app-mysql-local` on `127.0.0.1:3306`. Never cloud DB proxies or remote staging/prod.
2. **Do not** print DB passwords in chat, HTML reports, or PR/issue comments.
3. Parse local env files with **Python** — never `source` in zsh.
4. Write seed metadata to `/tmp/<case>-seed.json` (no secrets).
5. Deep INSERT / DBeaver patterns → also `mysql-insert`. Long recipes → [reference.md](reference.md).

## When to use / skip

| Use | Skip |
|-----|------|
| Start / ping / inspect local MySQL | Unit tests with mocks only |
| Seed or verify rows for a local API case | Remote staging unless user overrides |
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
- [ ] 2. Start/verify app-mysql-local (mysqladmin ping)
- [ ] 3. Align service local env to 127.0.0.1:3306
- [ ] 4. Schema present (dump or minimal CREATE) + discover FKs by real keys
- [ ] 5. Seed (prefer INSERT…SELECT via mysql-insert) + verify SELECT
- [ ] 6. Write /tmp/<case>-seed.json
- [ ] 7. Cleanup DELETE only if user asks
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

## Seed + verify (summary)

- Discover FK ids with `SELECT … WHERE <unique_name_column> = …` — do not invent UUIDs.
- Tag seed rows (`remarks` / equivalent) as `seed:<ticket-or-case>` when the schema allows.
- Verify with ticket-scoped queries, e.g. `WHERE remarks LIKE 'seed:<case>%' OR id LIKE '<case>%'`.
- Prefer **pymysql** if Homebrew `mysql` fails on auth plugin.

## Ticket-scoped inspect

```bash
docker exec app-mysql-local mysql -uapp -papp_local app_local -e \
  "SELECT * FROM <table> WHERE remarks LIKE 'seed:<TICKET>%' OR id LIKE '<ticket>%' LIMIT 100;"
```

Replace `<table>` / columns with the project schema.

## Related skills

| Skill | Role |
|-------|------|
| `mysql-insert` | INSERT…SELECT / DBeaver detail |
| `local-docker-redis` | Cache companion for API tests |
| `local-docker-dynamodb` | Document/key-value when the case needs it |
| `local-docker-firestore` | Firestore emulator when the case needs it |
| `playwright-local-api-test` | Local API probe + result report |

## Forbidden (unless user explicitly overrides)

- Cloud SQL / managed-DB proxies to remote staging
- Seeding staging or production hosts
- Committing real credentials into skill files or reports
